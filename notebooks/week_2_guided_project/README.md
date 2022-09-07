# Profitable App Profiles for the App Store and Google Play Markets
## :rocket: Introduction

Dataquest is an online course platform that has guided projects as a methodology for consolidating the knowledge passed on. In the Data Engineer path, the module [module name] has the guided project: Profitable App Profiles for the App Store and Google Play Markets. As requested in the discipline Algorithms and Data Structures II (DCA 0209) of the Computer Engineering course at the Federal University of Rio Grande do Norte, this repository contains the resolution of that project using the object-oriented programming (OOP) methodology.

## :clipboard: Dataset 

Let's say we're working as data analysts for a company that builds mobile apps for Android and iOS, and our job is to enable our team of developers to make data-driven decisions regarding the type of apps they build. Let's also assume that only free apps are created at our company and that our main source of income consists of in-app advertisements. This means that our company's revenue is directly related to the number of users of our app. So our goal for this project is to analyze data to help our developers understand what types of apps are likely to attract the most users.

As of September 2018, there were approximately 2 million iOS apps available on the App Store, and 2.1 million Android apps on Google Play.

![img](https://s3.amazonaws.com/dq-content/350/py1m8_statista.png) Source: [Statista](https://www.statista.com/statistics/276623/number-of-apps-available-in-leading-app-stores/)


Collecting data for over four million apps requires a significant amount of time and money, so we'll try to analyze a sample of data instead. To avoid spending resources with collecting new data ourselves, we should first try to see whether we can find any relevant existing data at no cost. Luckily, these are two data sets that seem suitable for our purpose:


 - A data set containing data about approximately ten thousand Android apps from Google Play. You can find this dataset on my [Github's repository](https://github.com/Morsinaldo/data_structure_II/tree/main/datasets)
 - A data set containing data about approximately seven thousand iOS apps from the App Store. You can find this dataset on my [Github's repository](https://github.com/Morsinaldo/data_structure_II/tree/main/datasets)
	After upload datasets we started creating Dataset class with following methods:

## :bulb: Solution
You can find the original solution on [Dataquest`s repository](https://github.com/dataquestio/solutions/blob/master/Mission350Solutions.ipynb) and my solution notebook on this [link](https://github.com/Morsinaldo/data_structure_II/blob/main/notebooks/week_2_guided_project/week_2_project.ipynbhttps://github.com/Morsinaldo/data_structure_II/blob/main/notebooks/week_2_guided_project/week_2_project.ipynb).
After upload datasets we started creating Dataset class with following methods:
- __init(self, filename)__: This function receives as a parameter the name of the file that will be opened, read and returned as a list.
- __explore_data(self, start, end, rows_and_columns=False)__: This function takes as parameters the start index, the end index and a boolean variable that informs if the user wants to view the dataset header. With this, the function will print the dataset lines between the start and end parameters, with no return.
- __check_duplicates(self)__: This function takes no parameters and checks for duplicate values ​​in the dataset. At the end, the function returns the names of repeated applications and the names of unique applications.
- __clean_data(self, dictionary)__: This function takes as a parameter a dictionary that will help the function keep repeating applications with the highest number of reviews.
- __clean_english(self, index)__: This function takes as a parameter the index of the column of application names and removes the lines whose names have characters that are not part of the English alphabet.
- __clean_free(self, column)__: This function takes as a parameter the column corresponding to the value of the applications and removes the rows whose price is different from zero.
- __freq_table(self, index)__: This function receives as a parameter the index of the column that wants to assemble the percentage table of applications.
- __display_table(self, index)__: This function receives as a parameter the index of the column that you want to view the application's frequency table.

### Importing libraries
The first step is to initialize the objects of the dataset class passing the name of the respective csv file as a parameter, as shown below.

```python
android = Dataset('googleplaystore.csv')
ios = Dataset('AppleStore.csv')
```

### Dataset exploration
We visualize some data using the `explore_data()` method.

```python
print(android.explore_data(0,3,True))
print(ios.explore_data(0,3,True))
```

### Deleting wrong data
Next, we delete the line corresponding to the Life Made WI-Fi Touchscreen Photo Fram app, as its rating is numbered 19, despite the maximum rating allowed is 5. The Google Play data set has a dedicated [discussion section](https://www.kaggle.com/lava18/google-play-store-apps/discussion), and you can see that [one of the discussions](https://www.kaggle.com/datasets/lava18/google-play-store-apps/discussion/66015) outlines an error for row 10473.

```python
del android.dataset[10473]
```

### Removing duplicates entries
Next, we assemble the dictionary to remove duplicate lines and then call the `clean_data()` method.

```python
reviews_max = {}
 
for app in android.dataset[1:]:
   name = app[0]
   n_reviews = float(app[3])
  
   if name in reviews_max and reviews_max[name] < n_reviews:
       reviews_max[name] = n_reviews
      
   elif name not in reviews_max:
       reviews_max[name] = n_reviews

android.clean_data(reviews_max)
```

### Removing non-English apps

We define a function called `is_english(string)` that checks whether the string received as a parameter belongs to the English alphabet or not. It will be used by the `clean_english()` method of the Dataset class to filter the rows of the two datasets.
```python
def is_english(string):
   non_ascii = 0
  
   for character in string:
       if ord(character) > 127:
           non_ascii += 1
  
   if non_ascii > 3:
       return False
   else:
       return True
android.clean_english(0)
print(android.explore_data(0,3,True))
ios.clean_english(1)
print(ios.explore_data(0,3,True))
```

### Isolating the free apps

We use the `clean_free()` method to filter the free applications in the two datasets.
```python
# Android price column index is 7
android.clean_free(7)
# Ios price column index is 4
ios.clean_free(4)
```

## :bar_chart: Conclusion
After performing all these steps, the frequency tables of several columns of the datasets were printed and the results were compared. You can find this information in the project notebook. With this we concluded that taking a popular book (perhaps a more recent book) and turning it into an app could be profitable for both the Google Play and the App Store markets. The markets are already full of libraries, so we need to add some special features besides the raw version of the book. This might include daily quotes from the book, an audio version of the book, quizzes on the book, a forum where people can discuss the book, etc.

## :books: References
- Dataquest - Data Engineer Path - [Python for Data Engineering: Fundametals Part II](https://www.dataquest.io/course/python-fundamentals-de-ii/) - Guided Project: Profitable App Profiles for the App Store and Google Play Markets
- Ivanovitch's repository ([Github](https://github.com/ivanovitchm/datastructure))