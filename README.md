## Webscraping Indeed.com


# Project Workflow: 
1. Ask an interesting question: 
2. Obtain the data
3. Explore the data
4. Communicate and visualize the results


### 1. Ask as interesting question: 
"Is the Iowa State University ABE program teaching the right skills that are needed for ABE careers on Indeed.com?"

### 2. Obtain the data:
I want to find keywords that are used in the Indeed.com and ISU ABE course catalogue: 
  - I will be using the Indeed.com homepage for my career keyword data.
  - I will also be using the ISU ABE course catalogue for my course key word data. 

### 3. Exploring the data:
My plan is to use a variety of webscraping functions (beautiful soup, nltk, requests) to explore what kind of data I can obtain from the two websites. 
Then, using nltk, I will be able to use seaborn to plot the frequencies of each keyword 

### 4. Communicate and visualize the results:

#### Auxilliary Function(s): 

```yml
# code needed to extract the name of the df

def get_df_name(df):
    name =[x for x in globals() if globals()[x] is df][0]
    return name

```

#### User Defined Function(s): 

```yml
# Course Catalogue Function:

from bs4 import BeautifulSoup
import requests
import pandas as pd

def course_cat(URL):
    headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15'}
    r = requests.get(URL, headers = headers)
    soup = BeautifulSoup(r.text, 'html.parser')
    df_courses = pd.DataFrame(columns = ["Descriptions"])
    descriptions = soup.find_all(class_ = "prereq")
    for description in descriptions:
        description_list = description.text
        df_courses.loc[len(df_courses.index)] = [description_list]
    df_course_names = pd.DataFrame(columns = ["Names"])
    names = soup.find_all(class_ = "toggle-accordion")
    for name in names:
        coursename_list = name.text
        df_course_names.loc[len(df_course_names.index)] = [coursename_list]
    compiled = pd.concat([df_course_names, df_courses], axis=1, join='inner')
    return compiled

```


### Course Catelogue Visual Function:
```yml
import seaborn as sns
import matplotlib.pyplot as plt
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.tokenize import sent_tokenize

def course_cat_figure(compiled):
    final_list = []

    stop_words = stopwords.words('english')
    newStopWords = ['department','curriculum', 'within', 'permission', 'introduction', 'credits','216Engineering', 'equivalent8', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
    stop_words.extend(newStopWords)
    for row in compiled.iterrows():
        words = word_tokenize(row[1]['Descriptions'])
        for word in words:
            if word.lower() not in stop_words:
                if len(word) > 5:
                    final_list.append(word)
                
    text = nltk.Text(final_list)
    all_fdist = nltk.FreqDist(text).most_common(30)
    all_fdist = pd.Series(dict(all_fdist))
    fig, ax = plt.subplots(figsize=(20,10))




    name = get_df_name(compiled)
    bar_plot = sns.barplot(x=all_fdist.values, y=all_fdist.index, orient='h', ax=ax)
    plt.title('Frequencies of the Most Common Words in the ' + name + ' Course Catalogue \n (12/04/21) \n', fontsize = 24)
    plt.xlabel('Frequency', fontsize=18)
    plt.ylabel('', fontsize=18)
    
    
    
 ```
