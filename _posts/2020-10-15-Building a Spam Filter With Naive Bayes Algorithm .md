---
layout: post
title: Building a Spam Filter With Naive Bayes
permlink: /Naive_Bayes_Algorithm
---


## Project Introduction:
In this project we're going to apply Naive Bayes algorithm by building a spam filter for SMS messages. Our goal is to create a program to classifies new messages with accuracy of greater than 80%. To train the algorithm, we'll use a dataset of 5,572 SMS messages that was put together by Tiago A. Almeida and José María Gómez Hidalgo from UCI Machiine Learning Repository. The data collection process is described in details on [this page](http://www.dt.fee.unicamp.br/~tiago/smsspamcollection/#composition).


Naive Bayes algorithm have multiple implementations in marketing area and it is commonly used in text classification.(eg:customer review classifications, sentiment analysis for social media responses)  Although we are not going to use dive into marketing use implementations for this project, the general methodlogies can be applied in simmilar environments. 


## Part 1 Exploring the Dataset
Let's stary by reading in the dataset


```python
import pandas as pd

sms_spam = pd.read_csv('SMSSpamCollection', sep='\t', header=None, names=['Label', 'SMS'])

print(sms_spam.shape)
sms_spam.head()
```

    (5572, 2)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Label</th>
      <th>SMS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Go until jurong point, crazy.. Available only ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>Ok lar... Joking wif u oni...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Free entry in 2 a wkly comp to win FA Cup fina...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>U dun say so early hor... U c already then say...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>Nah I don't think he goes to usf, he lives aro...</td>
    </tr>
  </tbody>
</table>
</div>



In the label column, ham means 'good mail'and spam labels indicate 'non-good'. from the small sample


```python
sms_spam['Label'].value_counts(normalize=True)
```




    ham     0.865937
    spam    0.134063
    Name: Label, dtype: float64



From our sample, We can see there are 13% of the messages are spam and 87% of the messages are useful information. 

## Part 2 Training and Test Set
The next step is to think of the way of testing and how well it works before we creating a spam filter. We're going to split the dataset into training and test set, where training dataset accounts for 80% of the data and the test set for the remaining 20%. 



```python
# Randomize the dataset
data_randomized = sms_spam.sample(frac=1, random_state=1)

# Calculate index for split
training_test_index = round(len(data_randomized) * 0.8)

# Training/Test split
training_set = data_randomized[:training_test_index].reset_index(drop=True)
test_set = data_randomized[training_test_index:].reset_index(drop=True)

print(training_set.shape)
print(test_set.shape)
```

    (4458, 2)
    (1114, 2)


We'll now analyze the percentage of spam and hame messages in the training and test sets. We expect the percentage to be similar to what we have in the full dataset, where about 87% of the messages are good messages, and remaining 13% are spam. 


```python
training_set['Label'].value_counts(normalize=True)
```




    ham     0.86541
    spam    0.13459
    Name: Label, dtype: float64




```python
test_set['Label'].value_counts(normalize=True)
```




    ham     0.868043
    spam    0.131957
    Name: Label, dtype: float64



## Part 3 Data Cleaning
To calculate all the probabilities required by the algorithm, we need to perform a little data clearning to bring the data in a format that allows us to extract all the information we need 


```python
#Before Data Cleaning
training_set.head()

#Remove the Punctuation marks / Replace Capital Letters
training_set['SMS'] = training_set['SMS'].str.replace('\W', ' ')
training_set['SMS'] = training_set['SMS'].str.lower()
training_set.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Label</th>
      <th>SMS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>yep  by the pretty sculpture</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>yes  princess  are you going to make me moan</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ham</td>
      <td>welp apparently he retired</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>havent</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>i forgot 2 ask ü all smth   there s a card on ...</td>
    </tr>
  </tbody>
</table>
</div>



## Part 3.1 Creating the Vocabulary
Vocabulary in this context means a list with all the unique words in our training set. 

<img src = "/public/images/Split SMS Column into Words Columns.jpg">


```python
training_set['SMS'] = training_set['SMS'].str.split()

vocabulary = []
for sms in training_set['SMS']:
    for word in sms:
        vocabulary.append(word)
## Remove the duplicated words in the list. 
vocabulary = list(set(vocabulary))
## Count unique words in the vocabulary
len(vocabulary)
```




    7783



## Part 3.2  Final Training Set
In order to transform the data format into the desired way, We'll first build a dictionary using the vocabulary we just created, then we'll then convert to the dataframe we need. 

* We start by initializing a dictionary named `word_counts_per_sms`, where each key is a unique word (a string) from the `vocabulary`, and each value is a list of the length of `training set`, where each element in the list is a 0.


```python
word_counts_per_sms = {unique_word: [0] * len(training_set['SMS']) for unique_word in vocabulary}

for index, sms in enumerate(training_set['SMS']):
    for word in sms:
        word_counts_per_sms[word][index] += 1
```

* We loop over `training_set['SMS']` using at the same time the `enumerate()` function to get both the `index` and the `SMS message` (index and sms).
     * Using a nested loop, we loop over sms (where sms is a list of strings, where each string represents a word in a message).
          * We incremenent `word_counts_per_sms[word][index]` by 1.


```python
# Conver the dictionary into dataframe
word_counts = pd.DataFrame(word_counts_per_sms)
word_counts.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>exe</th>
      <th>ymca</th>
      <th>1146</th>
      <th>09058094565</th>
      <th>speedchat</th>
      <th>bhaskar</th>
      <th>bpo</th>
      <th>gaytextbuddy</th>
      <th>grab</th>
      <th>heat</th>
      <th>...</th>
      <th>rgent</th>
      <th>910</th>
      <th>used</th>
      <th>cute</th>
      <th>kudi</th>
      <th>aint</th>
      <th>mgs</th>
      <th>pongal</th>
      <th>arun</th>
      <th>press</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 7783 columns</p>
</div>




```python
# Concatenate the Dataframes - training_set, word_counts
training_set_clean = pd.concat([training_set, word_counts], axis=1)
training_set_clean.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Label</th>
      <th>SMS</th>
      <th>exe</th>
      <th>ymca</th>
      <th>1146</th>
      <th>09058094565</th>
      <th>speedchat</th>
      <th>bhaskar</th>
      <th>bpo</th>
      <th>gaytextbuddy</th>
      <th>...</th>
      <th>rgent</th>
      <th>910</th>
      <th>used</th>
      <th>cute</th>
      <th>kudi</th>
      <th>aint</th>
      <th>mgs</th>
      <th>pongal</th>
      <th>arun</th>
      <th>press</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>[yep, by, the, pretty, sculpture]</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>[yes, princess, are, you, going, to, make, me,...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ham</td>
      <td>[welp, apparently, he, retired]</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>[havent]</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>[i, forgot, 2, ask, ü, all, smth, there, s, a,...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 7785 columns</p>
</div>



## Part 4 Calculating Constants First

Now we're done with data cleaning and have a training set to work with, we can begin creating the spam filter. Recall that the Naive Bayes algorithm will need to know the probability value of the two equations below to be able to classify new messages. 

<img src = "public/image/Naive Bayes Equation Part 1.jpg">

To calculate $p(w^{i}|Spam)$ and $p(w^{i}|Ham)$ inside the formula above, recall that we need to use these equations: 

<img src = "public/image/Naive Bayes Equation Part 2.jpg">

Some of the terms in the four equations above will have the same value for every new message. As a start, let's first caculate: 
* P(Spam) and P(Ham)
* $N_{spam}$, $N_{Ham}$, $N_{Vocabulary}$

We use laplace smoothing here and set $\alpha$ = 1
N_Spam is equal to the number of words in all the spam messages
N_Ham is equal to the number of words in all the non-spam messages


```python
# Isolating spam and ham messages first
spam_messages = training_set_clean[training_set_clean['Label'] == 'spam']
ham_messages = training_set_clean[training_set_clean['Label'] == 'ham']

# P(Spam) and P(Ham)
p_spam = len(spam_messages) / len(training_set_clean)
p_ham = len(ham_messages) / len(training_set_clean)

# N_Spam
n_words_per_spam_message = spam_messages['SMS'].apply(len)
n_spam = n_words_per_spam_message.sum()

# N_Ham
n_words_per_ham_message = ham_messages['SMS'].apply(len)
n_ham = n_words_per_ham_message.sum()

# N_Vocabulary
n_vocabulary = len(vocabulary)

# Laplace smoothing
alpha = 1
```

## Part 5 Calculating Parameters

Now that we have the constant terms calculated above, we can move on with calculating the parameters P($w_i|Spam$) and P($w_i|Ham$). Each parameter will thus be conditional probability value associated with each word in the vocabulary. 

The Parmeters are caculaed using the formulas:


```python
# Initiate parameters
parameters_spam = {unique_word:0 for unique_word in vocabulary}
parameters_ham = {unique_word:0 for unique_word in vocabulary}

# Calculate parameters
for word in vocabulary:
    n_word_given_spam = spam_messages[word].sum()   # spam_messages already defined in a cell above
    p_word_given_spam = (n_word_given_spam + alpha) / (n_spam + alpha*n_vocabulary)
    parameters_spam[word] = p_word_given_spam
    
    n_word_given_ham = ham_messages[word].sum()   # ham_messages already defined in a cell above
    p_word_given_ham = (n_word_given_ham + alpha) / (n_ham + alpha*n_vocabulary)
    parameters_ham[word] = p_word_given_ham
```

## Part 5.1 Classifying a New Message

now that we have all the parameters calculated, we can start creating the spam filter. The spam filter can be understood as a funtion that:
  * Takes in as input a new message (w1, w2, ..., wn)
  * Calculates P(Spam|w1, w2, ..., wn) and P(Ham|w1, w2, ..., wn)
  * Compares the values of P(Spam|w1, w2, ..., wn) and P(Ham|w1, w2, ..., wn), and:
     * If P(Ham|w1, w2, ..., wn) > P(Spam|w1, w2, ..., wn), then the message is classified as ham.
     * If P(Ham|w1, w2, ..., wn) < P(Spam|w1, w2, ..., wn), then the message is classified as spam.
     * If P(Ham|w1, w2, ..., wn) = P(Spam|w1, w2, ..., wn), then the algorithm may request human help.


```python
import re

def classify(message):
    '''
    message: a string
    '''
    
    message = re.sub('\W', ' ', message)
    message = message.lower().split()
    
    p_spam_given_message = p_spam
    p_ham_given_message = p_ham

    for word in message:
        if word in parameters_spam:
            p_spam_given_message *= parameters_spam[word]
            
        if word in parameters_ham:
            p_ham_given_message *= parameters_ham[word]
            
    print('P(Spam|message):', p_spam_given_message)
    print('P(Ham|message):', p_ham_given_message)
    
    if p_ham_given_message > p_spam_given_message:
        print('Label: Ham')
    elif p_ham_given_message < p_spam_given_message:
        print('Label: Spam')
    else:
        print('Equal proabilities, have a human classify this!')
```


```python
classify('WINNER!! This is the secret code to unlock the money: C3421')
```

    P(Spam|message): 1.3481290211300841e-25
    P(Ham|message): 1.9368049028589875e-27
    Label: Spam



```python
classify("Sounds good, Tom, then see u there")
```

    P(Spam|message): 2.4372375665888117e-25
    P(Ham|message): 3.687530435009238e-21
    Label: Ham


## Part 6 Measuring the Spam Filter's Accuracy

we managed to create a spam filter, and we classified two new messages. We'll now try to determine how well the spam filter does on our test set of 1,114 messages.


```python
def classify_test_set(message):    
    '''
    message: a string
    '''
    
    message = re.sub('\W', ' ', message)
    message = message.lower().split()
    
    p_spam_given_message = p_spam
    p_ham_given_message = p_ham

    for word in message:
        if word in parameters_spam:
            p_spam_given_message *= parameters_spam[word]
            
        if word in parameters_ham:
            p_ham_given_message *= parameters_ham[word]
    
    if p_ham_given_message > p_spam_given_message:
        return 'ham'
    elif p_spam_given_message > p_ham_given_message:
        return 'spam'
    else:
        return 'needs human classification'
```

Now that we have a function that returns labels instead of printing them, we can use it to create a new column in our test set.


```python
test_set['predicted'] = test_set['SMS'].apply(classify_test_set)
test_set.head() 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Label</th>
      <th>SMS</th>
      <th>predicted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ham</td>
      <td>Later i guess. I needa do mcat study too.</td>
      <td>ham</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ham</td>
      <td>But i haf enuff space got like 4 mb...</td>
      <td>ham</td>
    </tr>
    <tr>
      <th>2</th>
      <td>spam</td>
      <td>Had your mobile 10 mths? Update to latest Oran...</td>
      <td>spam</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ham</td>
      <td>All sounds good. Fingers . Makes it difficult ...</td>
      <td>ham</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ham</td>
      <td>All done, all handed in. Don't know if mega sh...</td>
      <td>ham</td>
    </tr>
  </tbody>
</table>
</div>



# Conclusion:
In this project, we managed to build a spam filter for SMS messages using the multinomial Naive Bayes algorithm. The filter had an accuracy of 98.74% on the test set, which is an excellent result. We initially aimed for an accuracy of over 80%, but we managed to do way better than that.


```python
correct = 0
total = test_set.shape[0]
    
for row in test_set.iterrows():
    row = row[1]
    if row['Label'] == row['predicted']:
        correct += 1
        
print('Correct:', correct)
print('Incorrect:', total - correct)
print('Accuracy:', correct/total)
```

    Correct: 1100
    Incorrect: 14
    Accuracy: 0.9874326750448833

