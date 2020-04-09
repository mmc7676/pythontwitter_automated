# pythontwitter_automated
Adds loop to avoid API's 100 maximum return for UserTimeLine and custom Wordcloud.  Data ready to use in Tableau or further Python Analysis


Jupyter Notebook
automated_pytwit_lg_timeline_w_wordcloud
Last Checkpoint: 10 minutes ago
(autosaved)
Current Kernel Logo
Python 3 
File
Edit
View
Insert
Cell
Kernel
Widgets
Help

I add a loop to the Twitter API in the python-twitter library to avoid the maximum 100 limit for UserTimeline search
I also add custom wordcloud and cleaned text dataframe options¶
!pip install python-twitter
Requirement already satisfied: python-twitter in c:\users\sixer\anaconda\lib\site-packages (3.5)
Requirement already satisfied: future in c:\users\sixer\anaconda\lib\site-packages (from python-twitter) (0.17.1)
Requirement already satisfied: requests-oauthlib in c:\users\sixer\anaconda\lib\site-packages (from python-twitter) (1.3.0)
Requirement already satisfied: requests in c:\users\sixer\anaconda\lib\site-packages (from python-twitter) (2.22.0)
Requirement already satisfied: oauthlib>=3.0.0 in c:\users\sixer\anaconda\lib\site-packages (from requests-oauthlib->python-twitter) (3.1.0)
Requirement already satisfied: certifi>=2017.4.17 in c:\users\sixer\anaconda\lib\site-packages (from requests->python-twitter) (2019.6.16)
Requirement already satisfied: idna<2.9,>=2.5 in c:\users\sixer\anaconda\lib\site-packages (from requests->python-twitter) (2.8)
Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in c:\users\sixer\anaconda\lib\site-packages (from requests->python-twitter) (1.24.2)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in c:\users\sixer\anaconda\lib\site-packages (from requests->python-twitter) (3.0.4)
import twitter
import json
import pandas as pd
import numpy as np
Save twiiter api credentials from developer acc
import keys
Adding tweet_mode = 'extended' is how you avoid trancated text.
api = twitter.Api(consumer_key=keys.consumer_key,
  consumer_secret=keys.consumer_secret,
  access_token_key=keys.access_token,
  access_token_secret=keys.access_token_secret,
  cache=None,
  tweet_mode=  'extended')
No retweets, no replies, Not sure if trim_user makes a difference-set to false prior to discovering tweet_mode/extended
ID from this gets the last tweetid to set maxid for loop

# need to get first tweet id
opp1 = api.GetUserTimeline(screen_name='@realDonaldTrump', count=1,include_rts=False, trim_user=False, exclude_replies=True)
opp1
[Status(ID=1248333612212195328, ScreenName=realDonaldTrump, Created=Thu Apr 09 19:35:00 +0000 2020, Text='The Wall Street Journal always “forgets” to mention that the ratings for the White House Press Briefings are “through the roof” (Monday Night Football, Bachelor Finale, according to @nytimes) &amp; is only way for me to escape the Fake News &amp; get my views across. WSJ is Fake News!')]
Set maxid = to ID then max_id=maxid
For what I am doing, it may also make sense to include entities because when I clean the text, it removes the hashtags from the full_text. Can use tableau to split

text is now full_text

I imagine this could be a lot cleaner, but I pretty much blunt forced this until I could get a loop to run for the tweet pull
ell[i] will give a json object for
1248319601647247361
maxid=1248333612212195328
ell=[] #ell[i] will give a json object for an individual tweet after run
for opp in range(0,50): # 30 seems arbitrary. 25 and 50 gave same results
    opp = api.GetUserTimeline(screen_name='@realDonaldTrump', count=100,max_id=maxid, include_rts=False, trim_user=False, exclude_replies=True)
    ilistnew = list(opp)
    for tweet in ilistnew:
        ell.append(json.loads(json.dumps(tweet._json)))
        maxid = ell[len(ell)-1].get("id") # I didn't know how to end this, which is why I set range
date = []
ids = []
text = []
favorite = []
retweet = []
entities = []
ex_entities=[]
for line in ell:
    date.append(line['created_at'])
    ids.append(line['id'])
    text.append(line['full_text'])
    favorite.append(line['favorite_count'])
    retweet.append(line['retweet_count'])
    entities.append(line['entities'])
​
newdf = pd.DataFrame(list(zip(date, ids,text,favorite,retweet,entities)), columns=['date', 'tweet_id','full_text','favorited','retweeted','entities'])
newdf['length'] = newdf.apply(lambda row: len(row.full_text), axis=1)
#newdf = newdf.drop_duplicates() # to keep entities-need to drop dups manually. if don't want entities, can remove and unhash
newdf.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1273 entries, 0 to 1272
Data columns (total 7 columns):
date         1273 non-null object
tweet_id     1273 non-null int64
full_text    1273 non-null object
favorited    1273 non-null int64
retweeted    1273 non-null int64
entities     1273 non-null object
length       1273 non-null int64
dtypes: int64(4), object(3)
memory usage: 69.7+ KB
Normally returns ~1300 tweets- to 12/19/19
newdf.tail()
date	tweet_id	full_text	favorited	retweeted	entities	length
1268	Fri Dec 27 14:16:44 +0000 2019	1210565179949404160	Thank you Kristy, have a great year! https://t...	74326	13539	{'hashtags': [], 'symbols': [], 'user_mentions...	60
1269	Fri Dec 27 13:48:31 +0000 2019	1210558076815912960	Academy Award winning actor (and great guy!) @...	75076	16569	{'hashtags': [], 'symbols': [], 'user_mentions...	283
1270	Fri Dec 27 04:27:44 +0000 2019	1210416954068078592	“Democrats repeatedly claimed impeachment was ...	85016	20093	{'hashtags': [], 'symbols': [], 'user_mentions...	209
1271	Fri Dec 27 04:20:25 +0000 2019	1210415111925649408	“Pelosi’s stall tactics expose the weakness of...	73197	16206	{'hashtags': [], 'symbols': [], 'user_mentions...	99
1272	Fri Dec 27 00:17:42 +0000 2019	1210354030997901312	Thank you Ritchie! https://t.co/1TnVNXRCdn	48814	11689	{'hashtags': [], 'symbols': [], 'user_mentions...	42
Export to CSV or Excel
ers
newdf.to_excel('tweeters.xlsx',header=True)
If want to clean the full_text column to remove links/non varchar
This removes many full tweets for this user-this is where having the entities can supplement the meaning of the hashtags and mentions removed

!pip install tweet-preprocessor
Collecting tweet-preprocessor
  Using cached tweet-preprocessor-0.5.0.tar.gz (6.3 kB)
    ERROR: Command errored out with exit status 1:
     command: 'c:\users\sixer\anaconda\python.exe' -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'C:\\Users\\sixer\\AppData\\Local\\Temp\\pip-install-uddla2jr\\tweet-preprocessor\\setup.py'"'"'; __file__='"'"'C:\\Users\\sixer\\AppData\\Local\\Temp\\pip-install-uddla2jr\\tweet-preprocessor\\setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base 'C:\Users\sixer\AppData\Local\Temp\pip-install-uddla2jr\tweet-preprocessor\pip-egg-info'
         cwd: C:\Users\sixer\AppData\Local\Temp\pip-install-uddla2jr\tweet-preprocessor\
    Complete output (7 lines):
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "C:\Users\sixer\AppData\Local\Temp\pip-install-uddla2jr\tweet-preprocessor\setup.py", line 6, in <module>
        long_description = f.read()
      File "c:\users\sixer\anaconda\lib\encodings\cp1252.py", line 23, in decode
        return codecs.charmap_decode(input,self.errors,decoding_table)[0]
    UnicodeDecodeError: 'charmap' codec can't decode byte 0x8d in position 652: character maps to <undefined>
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
!pip install nltk
Requirement already satisfied: nltk in c:\users\sixer\anaconda\lib\site-packages (3.4.4)
Requirement already satisfied: six in c:\users\sixer\anaconda\lib\site-packages (from nltk) (1.13.0)
import nltk
import re
nltk.download('stopwords')
from nltk.tokenize import RegexpTokenizer
from nltk.corpus import stopwords
import preprocessor as p
[nltk_data] Downloading package stopwords to
[nltk_data]     C:\Users\sixer\AppData\Roaming\nltk_data...
[nltk_data]   Package stopwords is already up-to-date!
def fix_Text(text):
    stop_words = set(stopwords.words("english"))
    custom = ['amp','https','kag','co']
    custom_list = stop_words.union(custom)
    letters = re.sub("[^a-zA-Z]"," ", str(text))
    words=letters.lower().split()
    goodwords=[word for word in words if word not in custom_list]
    return(" ".join(goodwords))
clean = newdf.full_text.apply(fix_Text)
clean.isnull().sum()
0
newdf['clean_text'] = clean
newdf.isnull().values.any()
False
!pip install wordcloud
Requirement already satisfied: wordcloud in c:\users\sixer\anaconda\lib\site-packages (1.6.0)
Requirement already satisfied: pillow in c:\users\sixer\anaconda\lib\site-packages (from wordcloud) (6.1.0)
Requirement already satisfied: numpy>=1.6.1 in c:\users\sixer\anaconda\lib\site-packages (from wordcloud) (1.18.1)
Requirement already satisfied: matplotlib in c:\users\sixer\anaconda\lib\site-packages (from wordcloud) (3.1.0)
Requirement already satisfied: cycler>=0.10 in c:\users\sixer\anaconda\lib\site-packages (from matplotlib->wordcloud) (0.10.0)
Requirement already satisfied: kiwisolver>=1.0.1 in c:\users\sixer\anaconda\lib\site-packages (from matplotlib->wordcloud) (1.1.0)
Requirement already satisfied: pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.1 in c:\users\sixer\anaconda\lib\site-packages (from matplotlib->wordcloud) (2.4.0)
Requirement already satisfied: python-dateutil>=2.1 in c:\users\sixer\anaconda\lib\site-packages (from matplotlib->wordcloud) (2.8.1)
Requirement already satisfied: six in c:\users\sixer\anaconda\lib\site-packages (from cycler>=0.10->matplotlib->wordcloud) (1.13.0)
Requirement already satisfied: setuptools in c:\users\sixer\anaconda\lib\site-packages (from kiwisolver>=1.0.1->matplotlib->wordcloud) (41.0.1)
from textblob import TextBlob
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import os
from os import path
from PIL import Image
Custom Wordcloud frrom stenciled image
d = path.dirname(__file__) if "__file__" in locals() else os.getcwd()
trump_mask = np.array(Image.open(path.join(d, "trumpcloud.png")))
wc = " ".join(item for item in clean)
wordcloud = WordCloud(background_color="black",mask=trump_mask).generate(wc)
plt.figure(figsize = (20, 20))
plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()

Create Dataframe with cleaned tweet column

df_clean = pd.DataFrame(list(zip(date, ids,text,clean,favorite,retweet)), columns=['date', 'tweet_id','full_text','clean','favorited','retweeted'])
Finally Export Dataframe with clean text to CSV
ers
df_clean.to_csv('clean_tweeters.csv',header=True)
