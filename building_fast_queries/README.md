# Building Fast Queries on a CSV

## Introduction and Dataset
The current climate crisis is one of the biggest problems facing modern humanity. The current climate crisis is one of the biggest problems facing modern humanity. With this in mind, one dataset that can be found on Kaggle is the [Reddit Climate Change Dataset](https://www.kaggle.com/datasets/pavellexyr/the-reddit-climate-change-dataset) with some sentiment data present in the Reddit comments about climate change. Futhermore dataset has a type of datapoint (type), an id, a subreddit id, a subreddit name, timestamp of the comment's creation, permalink to the comment on Reddit, comment's body text and a comment's score.

Another characteristic of the dataset is that it has about 4 million and 600 thousand lines, which was interesting for the Data Structure II discipline project on algorithm complexity. Thus, the task is to explore the performance of three functions with two different implements: one slower and one faster. The three main functions are:

1.   Given an id of a message on Reddit, return all information about the message;
2.   Given a lower and upper bound of the "sentiment" column, return all messages with sentiment values between the lower and upper bounds;
3.   Given a parameter value, return two messages whose sum of the value of the "score" column is equal to the parameter. Return -1 if it doesn't exist.

You can check complete code on [notebook] in this paste(./note.ipynb)

This project seeks to adapt the guided project "Building Fast Queries on a CSV" from the *Algorithms Complexity* course of the Data Engineer path, made by [Dataquest](Dataquest.io). So, first we created a class called Climate that receives as a parameter the name of the csv file to be initialized, as shown in the code below:

```python
class Climate():
    def __init__(self, filename):
        with open(filename) as f: 
            reader = csv.reader(f)
            rows = list(reader)
        
        # Separate the header from the data.
        self.header = rows[0]        
        self.rows = rows[1:]

        # Ordering by sentiment
        self.dict_score = {}
        for row in self.rows:
          if row[8] != '':      
            row[8] = float(row[8])                  
          else:
            row[8] = 0.0
          self.dict_score[int(row[9])] = row
        self.sentiment_to_rows = sorted(self.rows, key=row_sentiment)

        self.id_to_row = {}                        
        for row in self.rows:                       
            self.id_to_row[row[1]] = row
```

During class initialization, we separate the header and lines and store them in variables. After that we create a dict_score to store score values and transform the values of column 8 (sentiment) to float.

## Function 1 - Find an ID
For the first function, we implement a method that will iterate through all the rows comparing the desired id with the id of the current row of the iteration. So, in the worst case, this implementation will have an O(n) complexity, where n is the size of the vector The code of this implement can be shown below:

```python
def get_message_from_id(self, comment_id):
        '''
          Description: This function will iterate over all rows comparing the value of 
          column 1 (id) with the id fetched. If the corresponding id is found, 
          the function will return the complete line. If no id is found, the 
          function will return None.

          Parameters:
          - id <int>: value of the id we want to search

          Return: it returns the row with the respective id, or 
          returns None if there isn't this id in the dataset.
        '''
        for row in self.rows:                 
            if row[1] == comment_id:
                return row
        return None
```
The second implement method uses hashtable (or a dictionary in python) and has a complexity close to O(1) to serach an element.

```python
def get_message_from_id_fast(self, comment_id):  
        '''
        Description: This function will check if the searched id is in the 
        dictionary. If found, the function will return the corresponding line, 
        if not found, the function will return None.

        Parameters:
        - id <int>: value of the id we want to search

        Return: it returns the row with the respective id, or 
        returns none if there isn't this id in the dataset.
        '''
        if comment_id in self.id_to_row:           
            return self.id_to_row[comment_id]
        return None
```
**How to use:** You can instantiate an object from Climate class and call one of this methods above passing the desired id.
```python
# instantiate an object of class Climate
climate = Climate(filename)

# call the methods
print(climate.get_message_from_id('imlc8w9'))
print(climate.get_message_from_id_fast("d1e2748"))
```

## Function 2 - Getting the rows with the sentiment between a range
For the second function, we implement a method called `find_rows_by_sentiment` which, given a lower bound and an upper bound, returns the rows with the sentiment value between those bounds. For this, we implement a helper method called `find_menssage_range that`, given a sentiment value, performs a binary search on the `sentiment_to_rows` vector and returns the sentiment index passed as a parameter.

```python
def find_menssage_range(self, sentiment, rs=0):
        '''
        Description: This function performs a binary search for the index passed
        as a parameter and returns the index corresponding to the sentiment 
        value. If the searched value is not found, the function will return -1.

        Parameters:
        - sentiment <int>: value of the sentiment we want to search
        - rs <int>: index number of range start

        Return: If the index is found, the function returns it through the 
        range_middle variable. If not found, the function returns -1.
        '''
        range_start = rs
        range_end = len(self.sentiment_to_rows) - 1

        while range_start < range_end:
          range_middle = (range_end + range_start) // 2
          value = self.sentiment_to_rows[range_middle][8]
          if value == sentiment:
            return range_middle
          elif value < sentiment:
            range_start = range_middle + 1
          else:
            range_end = range_middle - 1
        if self.sentiment_to_rows[range_start][8] != sentiment:
          return -1
        return range_start

    def find_rows_by_sentiment(self, inf, sup):
        '''
        Description: This function receives as a parameter the lower and upper 
        limits of the sentiments to be searched and will call the 
        find_message_range function to find the indexes corresponding to the 
        searched values and will return the lines between the found indexes.

        Parameters:
        - inf <int>: inferior value of sentiment to be found
        - sup <int>: valor superior do sentimento a ser encontrado

        Return: If the indexes are found, the function returns lines between
        the range.
        '''
        inferior_limit = self.find_menssage_range(inf,0)
        if inferior_limit == -1:
          return -1

        superior_limit = self.find_menssage_range(sup, inferior_limit)
        if superior_limit == -1:
          return -1

        data = self.sentiment_to_rows[inferior_limit:superior_limit+1]
        return [row[7] for row in data]
```
**How to use:** You can instantiate an object from Climate class and call the method above passing the lower bound and the upper bound.
```python
# instantiate an object of class Climate
climate = Climate(filename)

# call the method
print(climate.find_rows_by_sentiment(0.283, 0.285))
```

## 3 - Find sum of scores
For the third function, we implement a method called `check_sum_sentiment()` that, given a number, the function will look for two lines whose sum of the scores of the lines is equal to the number informed as a parameter of the function.So, in the worst case, this implementation will have an O(n^2) complexity, where n is the size of the vector The code of this implement can be shown below:

```python
def check_sum_sentiments(self, target_sentiment):
    '''
    Description: This function will iterate over all rows comparing if the 
    sum of the scores of these two rows match the desired sum.

    Parameters:
    - target_sentiment <int>: Value whose sum of two scores is equal to it

    Return: If the sum is found, the function returns the corresponding 
    rows. If not found, the function returns -1.
    '''
    for row1 in self.rows:
      for row2 in self.rows:
        if int(row1[8]) + int(row2[8]) == target_sentiment:
          return row1,row2
    return -1
```
The second implement method uses hashtable (or a dictionary in python) and has a complexity close to O(1) to serach an element.

```python
def check_sum_sentiments_fast(self, target_sentiment):
    ''' Description: This function loops through the keys of the dics_sent 
    dictionary, takes the fetched sum and subtracts from the value of the 
    current iteration. If the value resulting from the subtraction is in the
    dictionary, the function returns the current row of the iteration and 
    the row whose key corresponds to the subtraction.

    Parameters:
    - target_sentiment <int>: Value whose sum of two scores is equal to it

    Return: If the sum is found, the function returns the corresponding 
    rows. If not found, the function returns -1.
    '''
    for sent in self.dict_sent:
      r = sent - target_sentiment
      if r in self.dict_sent:
        return self.dict_sent[sent], self.dict_sent[r]
    return -1
```
**How to use:** You can instantiate an object from Climate class and call one of this methods above passing the desired id.
```python
# instantiate an object of class Climate
climate = Climate(filename)

# call the methods
print(climate.check_sum_sentiments(40))
print(climate.check_sum_sentiments_fast(40))
```

## Time Comparison
### Get Message From ID
To compare the performance of functions `get_message_from_id` and `get_message_from_id_fast` in the search for an id. A list was created with all the ids of the dataset, and then 100 ids were randomly selected to be searched, and thus, the execution time of the functions was computed.

```python
ids = []
for row in climate.rows:
  ids.append(row[1])

ids_to_search = [ids[random.randint(0, len(ids))] for _ in range(100)]

total_time_no_dict = 0                                           
for i in ids_to_search:                                             
    start = time()                                            
    climate.get_message_from_id(i)                        
    end = time()                                             
    total_time_no_dict += end - start                              
    
total_time_dict = 0                                                 
for i in ids_to_search:                                              
    start = time()                                         
    climate.get_message_from_id_fast(i)               
    end = time()                                          
    total_time_dict += end - start                            
    
print('Total time without using dict: ', round(total_time_no_dict, 6))                                      
print('Total time using dict: ', round(total_time_dict, 6))
```
The results are: 
  - Total time without using dict:  28.320701
  - Total time using dict:  0.000149

It's possible to observe that the function `get_message_from_id_fast` (implemented with a dictonary) was much faster than the function `get_message_from_id` (implemented with for), in fact, it was 28.320701/0.000149 = 190.072 times faster.

### Get messages with sum scores
To compare the performance of functions `check_sm_sentiments` and `check_sum_sentiments_fast` in the search for an id. A score list was created with 10000 randomly generated numbers to be found, and thus, the execution time of the functions was computed.

```python
scores_to_search = [random.randint(0,200) for _ in range(10000)]

total_time_no_dict = 0                                           
for i in scores_to_search:                                             
    start = time()                                            
    climate.check_sum_sentiments(i)                        
    end = time()                                             
    total_time_no_dict += end - start                              
    
total_time_dict = 0                                                 
for i in scores_to_search:                                              
    start = time()                                         
    climate.check_sum_sentiments_fast(i)               
    end = time()                                          
    total_time_dict += end - start                            
    
print('Total time without using dict: ', round(total_time_no_dict, 6))                                      
print('Total time using dict: ', round(total_time_dict, 6))
```

The results are:
  - Total time without using dict:  39.967536
  - Total time using dict:  0.005891

It's possible to observe that the function `check_sum_sentiments_fast` (implemented with a dictonary) was much faster than the function `check_sum_sentiments` (implemented with for), in fact, it was 39.967536/0.005891 = 6.784 times faster.

## Conclusion 
With that in mind, we can observe the time difference between implementations with different time complexities and we can see that implementations using dictionaries are much more efficient in searching for an element than iterating over all positions of the vector.