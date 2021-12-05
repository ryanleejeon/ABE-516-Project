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
  - I would also like to use the course catalogue for other top ABE schools (Purdue and Cornell) 

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



#### Indeed Job Postings:
### Course Catelogue Visual Function:
```yml
def IndeedPostings(URL_from_indeed):
    df = pd.DataFrame(columns = ["Job_Titles"])
    df2 = pd.DataFrame(columns = ["Company"])
    df3 = pd.DataFrame(columns = ["Location"])
    df4 = pd.DataFrame(columns = ["Job_Description"])
    df5 = pd.DataFrame(columns = ["Salary (If Available)"])
    df6 = pd.DataFrame(columns = ["Link"])


    for pagenumber in range(0,500, 1):
        r = requests.get('https://www.indeed.com/jobs?q=agricultural%20engineering&start={}'.format(pagenumber), headers = headers)
        soup = BeautifulSoup(r.text, 'html.parser')
        titles = soup.select("h2 span") 
        # select all span tags under the umbrella of h2 tags 
        companies = soup.find_all(class_ = "companyName")
        locations = soup.find_all(class_ = "companyLocation")
        descriptions = soup.find_all(class_ = "job-snippet")
        salaries = soup.find_all(class_ = "salary-snippet")
        URLs = soup.find_all('a', attrs = {'class' : 'tapItem'})

        for title in titles:
            titles_list = title.text
            # the gotcha here is that there are "news" scattered so we have to remove those first before concatenating our data
            df.loc[len(df.index)] = [titles_list]
            df = df[df.Job_Titles != "new"]
        
        
        for company in companies:
            company_list = company.text
            df2.loc[len(df2.index)] = [company_list]
        
        
        for location in locations:
            location_list = location.text
            df3.loc[len(df3.index)] = [location_list]
        
    
        for description in descriptions:
            description_list = description.text
            df4.loc[len(df4.index)] = [description_list]
            df4['Job_Description'] = df4['Job_Description'].str.replace(r'\n', '', regex=True)
        # Another gotcha is that you have to remove the /n in each row- but looping can take considerable more time if we used hundreds of pages
    
        for salary in salaries:
            salary_list = salary.text
            df5.loc[len(df5.index)] = [salary_list]
        
    
        for URL in URLs:
            base = 'www.indeed.com'
            link = URL.attrs['href']
            new_URL = base + link
            df6.loc[len(df6.index)] = [new_URL]
    boom = pd.concat([df, df2, df3, df4, df5, df6], axis=1)
    return(boom)

```




#### Indeed Data Figures:
``` yml
def indeed_figure(indeed_table):
    final_list = []

    stop_words = stopwords.words('english')
    newStopWords = ['degree', 'experience', 'Experience', 'provide', 'and/or','including', 'related', 'located', 'Center', 'numerous','throughout', 'equivalent', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
    stop_words.extend(newStopWords)
    for row in indeed_table.iterrows():
        words = word_tokenize(row[1]['Job_Description'])
        for word in words:
            if word.lower() not in stop_words:
                if len(word) > 5:
                    final_list.append(word)
                
    text = nltk.Text(final_list)
    all_fdist = nltk.FreqDist(text).most_common(30)
    all_fdist = pd.Series(dict(all_fdist))
    fig, ax = plt.subplots(figsize=(20,10))




    bar_plot = sns.barplot(x=all_fdist.values, y=all_fdist.index, orient='h', ax=ax)
    plt.title('Frequencies of the Most Common Words in Indeed.com ABE Job Postings \n (12/04/21) \n', fontsize = 24)
    plt.xlabel('Frequency', fontsize=18)
    plt.ylabel('', fontsize=18)
 ```
