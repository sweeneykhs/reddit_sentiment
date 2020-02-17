```python
import praw
import pandas as pd
import datetime as dt
from praw.models import MoreComments

from bs4 import BeautifulSoup
import requests
import itertools
import time
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
url='https://redditmetrics.com/top'
response = requests.get(url)
soup = BeautifulSoup(response.content, "html.parser")
table = soup.find("table")
rows = table.find_all("tr")
top_subs = []
for row in rows:
        cells = row.find_all("td")
        cells = cells[0:11]
        top_subs.append([cell.text for cell in cells])

top_subs=pd.DataFrame(top_subs)
top_subs.columns=['Rank','Subreddit','Subscribers']
top_subs=top_subs.drop(0)
top_subs=top_subs.reset_index(drop=True)
i=0
for i in range(len(top_subs['Rank'])):
    top_subs['Subreddit'][i]=top_subs['Subreddit'][i][3:]
    i=+1

csv = top_subs
export_csv = csv.to_csv (r'~/Downloads/top_reddit.csv', index = None, header=True)
print (export_csv)
```

    None
    


```python
reddit = praw.Reddit(client_id='sAQk2I8qrhbCrQ', \
                     client_secret='xOcaeAZO5VjEQzBE5gx-HnUU9mk', \
                     user_agent='positivity', \
                     username='sweeney_khs', \
                     password='Flanker7')

```


```python
topics_data=pd.DataFrame([])
a=0
for a in range(len(top_subs['Rank'])):
    print(a)
    target=top_subs['Subreddit'][a]
    subreddit = reddit.subreddit(target)
    subreddit = subreddit.hot()
    subreddit = subreddit.hot(limit=500)
    topics_dict = { "title":[], \
                "score":[], \
                "id":[], "url":[], \
                "comms_num": [], \
                "created": [], \
                "body":[]}
    for submission in subreddit:
        topics_dict["title"].append(submission.title)
        topics_dict["score"].append(submission.score)
        topics_dict["id"].append(submission.id)
        topics_dict["url"].append(submission.url)
        topics_dict["comms_num"].append(submission.num_comments)
        topics_dict["created"].append(submission.created)
        topics_dict["body"].append(submission.selftext)
    interim=pd.DataFrame(topics_dict)
    interim['subreddit']=interim['title']
    b=0
    for b in range(len(interim['subreddit'])):
        interim['subreddit'][b]=target
        b=+1
    topics_data = topics_data.append(interim)
    
    a=+1
topics_data=topics_data.reset_index(drop=True)
csv = pd.DataFrame(topics_data)
export_csv = csv.to_csv (r'~/Downloads/topics_1.csv', index = None, header=True)
print (export_csv)
```

    0
    


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-45-8517eb57ab90> in <module>
          6     subreddit = reddit.subreddit(target)
          7     subreddit = subreddit.hot()
    ----> 8     subreddit = subreddit.hot(limit=500)
          9     topics_dict = { "title":[], \
         10                 "score":[], \
    

    AttributeError: 'ListingGenerator' object has no attribute 'hot'



```python
def get_date(created):
        return dt.datetime.fromtimestamp(created)
        _timestamp = topics_data["created"].apply(get_date)
        topics_data = topics_data.assign(timestamp = _timestamp)
topics_data['comments']=topics_data['title']
topics_data['top_comments']=topics_data['title']

```


```python
# start=time.time()
# i=0
# i_int=i-1
# while i <= len(topics_data['id']):
#     if i>0 and i%50==0:
#         safe=topics_data
#         end=time.time()
#         now=end-start
#         per_it=now/(i-i_int)
#         remaining=len(topics_data['id'])-i_int
#         time_remaining = (remaining *per_it)/3600
#         print('Done:',i,'Time per post:',per_it, 'Estimated Time Remaining:',time_remaining,'hours')
#     post=topics_data['id'][i]
#     submission = reddit.submission(id=post)
#     submission.comments.replace_more(limit=0)
#     txt=""
#     for top_level_comment in submission.comments.list():
#         txt = txt+comment.body
#     topics_data['top_comments'][i]=txt
#     i=i+1
# csv = pd.DataFrame(topics_data)
# export_csv = csv.to_csv (r'~/Downloads/topics_2.csv', index = None, header=True)
# print (export_csv)
```


```python
start=time.time()
i=28700
i_int=i-1
while i <= len(topics_data['id']):
    if i>0 and i%100==0:
        safe=topics_data
        end=time.time()
        now=end-start
        per_it=now/(i-i_int)
        remaining=len(topics_data['id'])-i
        time_remaining = (remaining *per_it)/3600
        print('Done:',i,'Time per post:',per_it, 'Estimated Time Remaining:',time_remaining,'hours')
        csv = pd.DataFrame(topics_data)
        export_csv = csv.to_csv (r'~/Downloads/topics_safety.csv', index = None, header=True)
        print (export_csv)
    post=topics_data['id'][i]
    submission = reddit.submission(id=post)
    submission.comments.replace_more(limit=0)
    txt=""
    for comment in submission.comments.list():
        txt = txt+comment.body
    topics_data['comments'][i]=txt
#     txt_2=""
#     for top_level_comment in submission.comments.list():
#         txt_2 = txt_2+top_level_comment.body
#     topics_data['top_comments'][i]=txt_2
    i=i+1
csv = pd.DataFrame(topics_data)
export_csv = csv.to_csv (r'~/Downloads/topics_2.csv', index = None, header=True)
print (export_csv)
```

    Done: 28700 Time per post: 0.009972810745239258 Estimated Time Remaining: 0.057839532097180686 hours
    None
    

    C:\Users\kevin.sweeney\Anaconda3\lib\site-packages\ipykernel_launcher.py:22: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
    

    Done: 28800 Time per post: 1.4958735621801698 Estimated Time Remaining: 8.634099096817152 hours
    None
    Done: 28900 Time per post: 1.8242385683961175 Estimated Time Remaining: 10.478730376628699 hours
    None
    Done: 29000 Time per post: 1.9980936438538308 Estimated Time Remaining: 11.421880304685551 hours
    None
    Done: 29100 Time per post: 2.072365391581433 Estimated Time Remaining: 11.788880792832268 hours
    None
    Done: 29200 Time per post: 1.9637022027950326 Estimated Time Remaining: 11.116190886322213 hours
    None
    Done: 29300 Time per post: 1.8664443207263153 Estimated Time Remaining: 10.513784550002486 hours
    None
    Done: 29400 Time per post: 1.7785602663451017 Estimated Time Remaining: 9.969324337382723 hours
    None
    Done: 29500 Time per post: 1.6860664102170946 Estimated Time Remaining: 9.404035402985844 hours
    None
    Done: 29600 Time per post: 1.60102925792783 Estimated Time Remaining: 8.885267651150032 hours
    None
    Done: 29700 Time per post: 1.6031565130292833 Estimated Time Remaining: 8.852541200696978 hours
    None
    Done: 29800 Time per post: 1.5866750841461237 Estimated Time Remaining: 8.717457358146161 hours
    None
    Done: 29900 Time per post: 1.5592135241585507 Estimated Time Remaining: 8.523267483865588 hours
    None
    Done: 30000 Time per post: 1.5136438451484018 Estimated Time Remaining: 8.232120234489043 hours
    None
    Done: 30100 Time per post: 1.673250151395287 Estimated Time Remaining: 9.053677694174665 hours
    None
    Done: 30200 Time per post: 2.214137105763872 Estimated Time Remaining: 11.91882304794391 hours
    None
    Done: 30300 Time per post: 2.6258966047119006 Estimated Time Remaining: 14.062405733955758 hours
    None
    Done: 30400 Time per post: 2.928445506978918 Estimated Time Remaining: 15.601293438430185 hours
    None
    Done: 30500 Time per post: 3.1511469244758397 Estimated Time Remaining: 16.700203381131818 hours
    None
    Done: 30600 Time per post: 3.257510855472571 Estimated Time Remaining: 17.17341625722609 hours
    None
    Done: 30700 Time per post: 3.180134882038084 Estimated Time Remaining: 16.677157343888055 hours
    None
    Done: 30800 Time per post: 3.1033322907583534 Estimated Time Remaining: 16.18818808004198 hours
    None
    Done: 30900 Time per post: 3.0231683988670826 Estimated Time Remaining: 15.686045145121732 hours
    None
    Done: 31000 Time per post: 2.9384869110682694 Estimated Time Remaining: 15.165041200204826 hours
    None
    Done: 31100 Time per post: 2.8691672964028547 Estimated Time Remaining: 14.727595130618987 hours
    None
    Done: 31200 Time per post: 2.8194718846127014 Estimated Time Remaining: 14.394187157582456 hours
    None
    Done: 31300 Time per post: 2.7658952201893126 Estimated Time Remaining: 14.043832980511235 hours
    None
    Done: 31400 Time per post: 2.704492871031325 Estimated Time Remaining: 13.65693775068846 hours
    None
    Done: 31500 Time per post: 2.638750434389969 Estimated Time Remaining: 13.251658084260068 hours
    None
    Done: 31600 Time per post: 2.603016048495172 Estimated Time Remaining: 12.999895982192973 hours
    None
    Done: 31700 Time per post: 2.564307124326325 Estimated Time Remaining: 12.73534640995288 hours
    None
    Done: 31800 Time per post: 2.5227605557680053 Estimated Time Remaining: 12.458933311388712 hours
    None
    Done: 31900 Time per post: 2.4776091037113868 Estimated Time Remaining: 12.167125373476003 hours
    None
    Done: 32000 Time per post: 2.4270819540783624 Estimated Time Remaining: 11.851576019650981 hours
    None
    Done: 32100 Time per post: 2.4032070353535193 Estimated Time Remaining: 11.668237714151156 hours
    None
    Done: 32200 Time per post: 2.3795564959846676 Estimated Time Remaining: 11.487308984365983 hours
    None
    Done: 32300 Time per post: 2.348094245520806 Estimated Time Remaining: 11.270200130098335 hours
    None
    Done: 32400 Time per post: 2.3111243389709935 Estimated Time Remaining: 11.028556949772971 hours
    None
    Done: 32500 Time per post: 2.275930436276599 Estimated Time Remaining: 10.797393311435565 hours
    None
    Done: 32600 Time per post: 2.259438407204146 Estimated Time Remaining: 10.656390198866442 hours
    None
    Done: 32700 Time per post: 2.2473023423431098 Estimated Time Remaining: 10.536726732335932 hours
    None
    Done: 32800 Time per post: 2.227582635370239 Estimated Time Remaining: 10.382391399688123 hours
    None
    Done: 32900 Time per post: 2.2101690998476933 Estimated Time Remaining: 10.239836226766577 hours
    None
    Done: 33000 Time per post: 2.1968821309051303 Estimated Time Remaining: 10.117252457854487 hours
    None
    Done: 33100 Time per post: 2.179172064872851 Estimated Time Remaining: 9.975160126955474 hours
    None
    Done: 33200 Time per post: 2.1543783103855576 Estimated Time Remaining: 9.801822873834736 hours
    None
    Done: 33300 Time per post: 2.1259533892090334 Estimated Time Remaining: 9.613443117481626 hours
    None
    Done: 33400 Time per post: 2.1090526264846337 Estimated Time Remaining: 9.478434012193025 hours
    None
    Done: 33500 Time per post: 2.0954123020072797 Estimated Time Remaining: 9.358926223326401 hours
    None
    Done: 33600 Time per post: 2.0789403733465774 Estimated Time Remaining: 9.2276078404736 hours
    None
    Done: 33700 Time per post: 2.060359764804699 Estimated Time Remaining: 9.087903529259394 hours
    None
    Done: 33800 Time per post: 2.0370110061397226 Estimated Time Remaining: 8.928332407188524 hours
    None
    Done: 33900 Time per post: 2.026256147840119 Estimated Time Remaining: 8.824908372773674 hours
    None
    Done: 34000 Time per post: 2.0151998160627964 Estimated Time Remaining: 8.720777204011751 hours
    None
    Done: 34100 Time per post: 2.0074191438646323 Estimated Time Remaining: 8.631344702189068 hours
    None
    Done: 34200 Time per post: 2.0059425930525254 Estimated Time Remaining: 8.56927531626522 hours
    None
    Done: 34300 Time per post: 2.0081079069194443 Estimated Time Remaining: 8.522744641617274 hours
    None
    Done: 34400 Time per post: 2.004597934917951 Estimated Time Remaining: 8.452164459477661 hours
    None
    Done: 34500 Time per post: 1.9948553905427877 Estimated Time Remaining: 8.355673453887416 hours
    None
    Done: 34600 Time per post: 1.9923955887136895 Estimated Time Remaining: 8.29002597870621 hours
    None
    Done: 34700 Time per post: 1.9829394853109121 Estimated Time Remaining: 8.19559905609474 hours
    None
    Done: 34800 Time per post: 1.971460254558519 Estimated Time Remaining: 8.09339197281121 hours
    None
    Done: 34900 Time per post: 1.9572614778762594 Estimated Time Remaining: 7.9807336760404475 hours
    None
    Done: 35000 Time per post: 1.9416834368552127 Estimated Time Remaining: 7.863278562753374 hours
    None
    Done: 35100 Time per post: 1.9365129667862264 Estimated Time Remaining: 7.788547568360492 hours
    None
    Done: 35200 Time per post: 1.9274272981780691 Estimated Time Remaining: 7.698465866806238 hours
    None
    Done: 35300 Time per post: 1.916951315888057 Estimated Time Remaining: 7.603374399879324 hours
    None
    Done: 35400 Time per post: 1.9041263798140284 Estimated Time Remaining: 7.499613316495308 hours
    None
    Done: 35500 Time per post: 1.8952959786616885 Estimated Time Remaining: 7.412186689882754 hours
    None
    Done: 35600 Time per post: 1.8953558636167709 Estimated Time Remaining: 7.3597721159719 hours
    None
    Done: 35700 Time per post: 1.890218472654454 Estimated Time Remaining: 7.287317272769768 hours
    None
    Done: 35800 Time per post: 1.8792356864916844 Estimated Time Remaining: 7.1927745900469215 hours
    None
    Done: 35900 Time per post: 1.8727474600685055 Estimated Time Remaining: 7.115920140632523 hours
    None
    Done: 36000 Time per post: 1.8684436096718597 Estimated Time Remaining: 7.047665493259495 hours
    None
    Done: 36100 Time per post: 1.8638019409071447 Estimated Time Remaining: 6.978385100413167 hours
    None
    Done: 36200 Time per post: 1.859764655261592 Estimated Time Remaining: 6.911608700762455 hours
    None
    Done: 36300 Time per post: 1.8557143457279976 Estimated Time Remaining: 6.845008554700578 hours
    None
    Done: 36400 Time per post: 1.85437432457419 Estimated Time Remaining: 6.78855533987868 hours
    None
    Done: 36500 Time per post: 1.8503701582335155 Estimated Time Remaining: 6.722497583204485 hours
    None
    Done: 36600 Time per post: 1.8418578197195354 Estimated Time Remaining: 6.64040906726107 hours
    None
    Done: 36700 Time per post: 1.8295442903359194 Estimated Time Remaining: 6.5451946986767515 hours
    None
    Done: 36800 Time per post: 1.8255630656092745 Estimated Time Remaining: 6.480241782061366 hours
    None
    Done: 36900 Time per post: 1.8203535018614365 Estimated Time Remaining: 6.411183902805877 hours
    None
    Done: 37000 Time per post: 1.8134106930559364 Estimated Time Remaining: 6.3363591966529516 hours
    None
    Done: 37100 Time per post: 1.8043275340343286 Estimated Time Remaining: 6.254500915892885 hours
    None
    Done: 37200 Time per post: 1.7946164472259334 Estimated Time Remaining: 6.170988055613841 hours
    None
    Done: 37300 Time per post: 1.7910739720775533 Estimated Time Remaining: 6.109054806427855 hours
    None
    Done: 37400 Time per post: 1.7861731879917482 Estimated Time Remaining: 6.042723126819862 hours
    None
    Done: 37500 Time per post: 1.7793830300905313 Estimated Time Remaining: 5.9703243390176475 hours
    None
    Done: 37600 Time per post: 1.7705335522780137 Estimated Time Remaining: 5.891450395205091 hours
    None
    Done: 37700 Time per post: 1.762887271374229 Estimated Time Remaining: 5.817038304626241 hours
    None
    Done: 37800 Time per post: 1.7605740270854588 Estimated Time Remaining: 5.760500406955449 hours
    None
    Done: 37900 Time per post: 1.7581413831546013 Estimated Time Remaining: 5.703703670517386 hours
    None
    Done: 38000 Time per post: 1.7531642138656782 Estimated Time Remaining: 5.638857897875191 hours
    None
    Done: 38100 Time per post: 1.7501124408495907 Estimated Time Remaining: 5.580427974586793 hours
    None
    Done: 38200 Time per post: 1.763326078298882 Estimated Time Remaining: 5.573579845823049 hours
    None
    Done: 38300 Time per post: 1.7715536716666995 Estimated Time Remaining: 5.550376072980196 hours
    None
    Done: 38400 Time per post: 1.7848106724320583 Estimated Time Remaining: 5.542332918643883 hours
    None
    Done: 38500 Time per post: 1.7875558860982368 Estimated Time Remaining: 5.501203239467324 hours
    None
    Done: 38600 Time per post: 1.7914552335822935 Estimated Time Remaining: 5.463440835972222 hours
    None
    Done: 38700 Time per post: 1.7871757148921568 Estimated Time Remaining: 5.400745722864381 hours
    None
    Done: 38800 Time per post: 1.7817564874506862 Estimated Time Remaining: 5.334875882841929 hours
    None
    Done: 38900 Time per post: 1.774721270927599 Estimated Time Remaining: 5.264513458954397 hours
    None
    Done: 39000 Time per post: 1.7656408757471873 Estimated Time Remaining: 5.188531895702638 hours
    None
    Done: 39100 Time per post: 1.7651816693742326 Estimated Time Remaining: 5.138149642603496 hours
    None
    Done: 39200 Time per post: 1.7683212800702983 Estimated Time Remaining: 5.0981684905137845 hours
    None
    Done: 39300 Time per post: 1.7662373443289012 Estimated Time Remaining: 5.043098239543549 hours
    None
    Done: 39400 Time per post: 1.763244072640792 Estimated Time Remaining: 4.985572615391839 hours
    None
    Done: 39500 Time per post: 1.7641493748342667 Estimated Time Remaining: 4.939128208042937 hours
    None
    Done: 39600 Time per post: 1.7629140475815277 Estimated Time Remaining: 4.886699800226684 hours
    None
    Done: 39700 Time per post: 1.7581647365009359 Estimated Time Remaining: 4.824697064414652 hours
    None
    Done: 39800 Time per post: 1.7513305693958785 Estimated Time Remaining: 4.757294899478415 hours
    None
    Done: 39900 Time per post: 1.7510927393546905 Estimated Time Remaining: 4.708007395615014 hours
    None
    Done: 40000 Time per post: 1.747801383505162 Estimated Time Remaining: 4.6506081812766515 hours
    None
    Done: 40100 Time per post: 1.7441392822565085 Estimated Time Remaining: 4.592415626808179 hours
    None
    Done: 40200 Time per post: 1.7370833237578938 Estimated Time Remaining: 4.525584581534801 hours
    None
    Done: 40300 Time per post: 1.73356732468432 Estimated Time Remaining: 4.468269779373835 hours
    None
    Done: 40400 Time per post: 1.731475996977243 Estimated Time Remaining: 4.414782826737254 hours
    None
    Done: 40500 Time per post: 1.7282571705728556 Estimated Time Remaining: 4.358568569897487 hours
    None
    Done: 40600 Time per post: 1.7243986572300563 Estimated Time Remaining: 4.300937650907965 hours
    None
    Done: 40700 Time per post: 1.7199410453516983 Estimated Time Remaining: 4.242043483799369 hours
    None
    Done: 40800 Time per post: 1.71949836900634 Estimated Time Remaining: 4.193187828196294 hours
    None
    Done: 40900 Time per post: 1.7170573808005496 Estimated Time Remaining: 4.139539168879992 hours
    None
    Done: 41000 Time per post: 1.7131108054784476 Estimated Time Remaining: 4.082438222277667 hours
    None
    Done: 41100 Time per post: 1.7087797088859908 Estimated Time Remaining: 4.0246508754567545 hours
    None
    Done: 41200 Time per post: 1.7109479465519712 Estimated Time Remaining: 3.982231345599713 hours
    None
    Done: 41300 Time per post: 1.7111127017821672 Estimated Time Remaining: 3.935083905015156 hours
    None
    Done: 41400 Time per post: 1.7090352959974893 Estimated Time Remaining: 3.8828332461009625 hours
    None
    Done: 41500 Time per post: 1.704086981474557 Estimated Time Remaining: 3.8242552009258186 hours
    None
    Done: 41600 Time per post: 1.70406575247665 Estimated Time Remaining: 3.7768723997253306 hours
    None
    Done: 41700 Time per post: 1.7015931574017 Estimated Time Remaining: 3.7241256908799985 hours
    None
    Done: 41800 Time per post: 1.6981332864245242 Estimated Time Remaining: 3.669383009748993 hours
    None
    Done: 41900 Time per post: 1.6929025050049775 Estimated Time Remaining: 3.6110550933147842 hours
    None
    Done: 42000 Time per post: 1.6876939456002729 Estimated Time Remaining: 3.5530645593623524 hours
    None
    Done: 42100 Time per post: 1.689704500786467 Estimated Time Remaining: 3.5103611003838853 hours
    None
    Done: 42200 Time per post: 1.6975083950669418 Estimated Time Remaining: 3.47942067977749 hours
    None
    Done: 42300 Time per post: 1.7036872954502167 Estimated Time Remaining: 3.444761062106146 hours
    None
    Done: 42400 Time per post: 1.710027276647412 Estimated Time Remaining: 3.4100793941810474 hours
    None
    Done: 42500 Time per post: 1.7138431932207345 Estimated Time Remaining: 3.3700822124471057 hours
    None
    Done: 42600 Time per post: 1.7253275597303876 Estimated Time Remaining: 3.344739177599549 hours
    None
    Done: 42700 Time per post: 1.7280998116739732 Estimated Time Remaining: 3.3021107234736835 hours
    None
    Done: 42800 Time per post: 1.7322106688388332 Estimated Time Remaining: 3.2618489233495698 hours
    None
    Done: 42900 Time per post: 1.7345421077348708 Estimated Time Remaining: 3.2180574271003337 hours
    None
    Done: 43000 Time per post: 1.7341155870153673 Estimated Time Remaining: 3.1690962352705836 hours
    None
    Done: 43100 Time per post: 1.736352168903625 Estimated Time Remaining: 3.1249515839796076 hours
    None
    Done: 43200 Time per post: 1.7336900352601405 Estimated Time Remaining: 3.0720024263678987 hours
    None
    Done: 43300 Time per post: 1.730074607413988 Estimated Time Remaining: 3.0175384610978977 hours
    None
    Done: 43400 Time per post: 1.7248781400102797 Estimated Time Remaining: 2.960561674200977 hours
    None
    Done: 43500 Time per post: 1.7203299861672392 Estimated Time Remaining: 2.9049683294196242 hours
    None
    Done: 43600 Time per post: 1.7204710448094642 Estimated Time Remaining: 2.857415660254385 hours
    None
    Done: 43700 Time per post: 1.7219382317954164 Estimated Time Remaining: 2.812020795757015 hours
    None
    Done: 43800 Time per post: 1.7199773582888946 Estimated Time Remaining: 2.7610414315420893 hours
    None
    Done: 43900 Time per post: 1.7203688847533214 Estimated Time Remaining: 2.7138819156983645 hours
    None
    Done: 44000 Time per post: 1.7206175198127114 Estimated Time Remaining: 2.6664792063986438 hours
    None
    Done: 44100 Time per post: 1.719170203991319 Estimated Time Remaining: 2.6164815410190103 hours
    None
    Done: 44200 Time per post: 1.715802743322472 Estimated Time Remaining: 2.5636952656476604 hours
    None
    Done: 44300 Time per post: 1.7161964332054123 Estimated Time Remaining: 2.5166113808031585 hours
    None
    Done: 44400 Time per post: 1.7152352236948243 Estimated Time Remaining: 2.4675564509765264 hours
    None
    Done: 44500 Time per post: 1.7130092084653 Estimated Time Remaining: 2.416770491609794 hours
    None
    Done: 44600 Time per post: 1.7090752600394032 Estimated Time Remaining: 2.3637460332600524 hours
    None
    Done: 44700 Time per post: 1.70858520695913 Estimated Time Remaining: 2.315607562431554 hours
    None
    Done: 44800 Time per post: 1.7077106572287355 Estimated Time Remaining: 2.2669858974711463 hours
    None
    Done: 44900 Time per post: 1.7056068132324105 Estimated Time Remaining: 2.216815077531791 hours
    None
    Done: 45000 Time per post: 1.702052177033478 Estimated Time Remaining: 2.1649158107323045 hours
    None
    Done: 45100 Time per post: 1.6986249718649504 Estimated Time Remaining: 2.113372569161976 hours
    None
    Done: 45200 Time per post: 1.6976675254548292 Estimated Time Remaining: 2.0650239149907494 hours
    None
    Done: 45300 Time per post: 1.6957816368312364 Estimated Time Remaining: 2.0156248955557947 hours
    None
    Done: 45400 Time per post: 1.6925873175387425 Estimated Time Remaining: 1.9648117777762235 hours
    None
    Done: 45500 Time per post: 1.6881972561208387 Estimated Time Remaining: 1.9128212799213613 hours
    None
    Done: 45600 Time per post: 1.6863875191886428 Estimated Time Remaining: 1.8639266496810025 hours
    None
    Done: 45700 Time per post: 1.6877408654232193 Estimated Time Remaining: 1.818540782493519 hours
    None
    Done: 45800 Time per post: 1.6863291748336418 Estimated Time Remaining: 1.7701772088045369 hours
    None
    Done: 45900 Time per post: 1.6837696992937239 Estimated Time Remaining: 1.720719089917114 hours
    None
    Done: 46000 Time per post: 1.6816981476680728 Estimated Time Remaining: 1.6718882418066758 hours
    None
    Done: 46100 Time per post: 1.682024228491322 Estimated Time Remaining: 1.6254895252559194 hours
    None
    Done: 46200 Time per post: 1.680257799093958 Estimated Time Remaining: 1.57710863976069 hours
    None
    Done: 46300 Time per post: 1.6779609559293593 Estimated Time Remaining: 1.528342770692325 hours
    None
    Done: 46400 Time per post: 1.676830640681537 Estimated Time Remaining: 1.4807346129796128 hours
    None
    Done: 46500 Time per post: 1.6779872162892144 Estimated Time Remaining: 1.4351451774873587 hours
    None
    Done: 46600 Time per post: 1.6779245344009301 Estimated Time Remaining: 1.3884825522167696 hours
    None
    Done: 46700 Time per post: 1.6737424063487858 Estimated Time Remaining: 1.3385289966328207 hours
    None
    Done: 46800 Time per post: 1.668511778720472 Estimated Time Remaining: 1.2879983980733867 hours
    None
    Done: 46900 Time per post: 1.6679713645876584 Estimated Time Remaining: 1.2412486904806492 hours
    None
    Done: 47000 Time per post: 1.6670183785384707 Estimated Time Remaining: 1.1942334439585323 hours
    None
    Done: 47100 Time per post: 1.6690663691351164 Estimated Time Remaining: 1.1493376469683203 hours
    None
    Done: 47200 Time per post: 1.6783936980427758 Estimated Time Remaining: 1.1091385021232676 hours
    None
    Done: 47300 Time per post: 1.6830526214858215 Estimated Time Remaining: 1.0654658123239409 hours
    None
    Done: 47400 Time per post: 1.6864361535558776 Estimated Time Remaining: 1.0207623273884048 hours
    None
    Done: 47500 Time per post: 1.6897115141563888 Estimated Time Remaining: 0.9758083994253145 hours
    None
    Done: 47600 Time per post: 1.6897759869340727 Estimated Time Remaining: 0.9289074105951471 hours
    None
    Done: 47700 Time per post: 1.6884382270736498 Estimated Time Remaining: 0.8812709524087188 hours
    None
    Done: 47800 Time per post: 1.6885472939442692 Estimated Time Remaining: 0.8344237877574596 hours
    None
    Done: 47900 Time per post: 1.6875343324492285 Estimated Time Remaining: 0.7870472622728485 hours
    None
    Done: 48000 Time per post: 1.6851045658449937 Estimated Time Remaining: 0.7391055859636791 hours
    None
    Done: 48100 Time per post: 1.6824782602160648 Estimated Time Remaining: 0.6912181519054332 hours
    None
    Done: 48200 Time per post: 1.6839642467623974 Estimated Time Remaining: 0.6450518600792627 hours
    None
    Done: 48300 Time per post: 1.6842025881074874 Estimated Time Remaining: 0.59835975283041 hours
    None
    Done: 48400 Time per post: 1.6825767551531714 Estimated Time Remaining: 0.5510438873126636 hours
    None
    Done: 48500 Time per post: 1.6819306875165019 Estimated Time Remaining: 0.504112003286196 hours
    None
    Done: 48600 Time per post: 1.682312812285713 Estimated Time Remaining: 0.45749562311880915 hours
    None
    Done: 48700 Time per post: 1.6808068295382457 Estimated Time Remaining: 0.41039700087892167 hours
    None
    Done: 48800 Time per post: 1.6778772525375538 Estimated Time Remaining: 0.3630739943685429 hours
    None
    Done: 48900 Time per post: 1.6747137985514635 Estimated Time Remaining: 0.31586963033790105 hours
    None
    Done: 49000 Time per post: 1.6746520135127982 Estimated Time Remaining: 0.26933986550664174 hours
    None
    Done: 49100 Time per post: 1.6746568200658463 Estimated Time Remaining: 0.22282239355876118 hours
    None
    Done: 49200 Time per post: 1.6738155572555464 Estimated Time Remaining: 0.1762155822777367 hours
    None
    Done: 49300 Time per post: 1.6713026870044074 Estimated Time Remaining: 0.12952595824284158 hours
    None
    Done: 49400 Time per post: 1.6715134848178248 Estimated Time Remaining: 0.08311136493955294 hours
    None
    Done: 49500 Time per post: 1.6708117523168602 Estimated Time Remaining: 0.036665035675842214 hours
    None
    


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-42-cf084b4ee033> in <module>
         14         export_csv = csv.to_csv (r'~/Downloads/topics_safety_cont.csv', index = None, header=True)
         15         print (export_csv)
    ---> 16     post=topics_data['id'][i]
         17     submission = reddit.submission(id=post)
         18     submission.comments.replace_more(limit=0)
    

    ~\Anaconda3\lib\site-packages\pandas\core\series.py in __getitem__(self, key)
       1069         key = com.apply_if_callable(key, self)
       1070         try:
    -> 1071             result = self.index.get_value(self, key)
       1072 
       1073             if not is_scalar(result):
    

    ~\Anaconda3\lib\site-packages\pandas\core\indexes\base.py in get_value(self, series, key)
       4728         k = self._convert_scalar_indexer(k, kind="getitem")
       4729         try:
    -> 4730             return self._engine.get_value(s, k, tz=getattr(series.dtype, "tz", None))
       4731         except KeyError as e1:
       4732             if len(self) > 0 and (self.holds_integer() or self.is_boolean()):
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()
    

    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()
    

    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()
    

    KeyError: 49579



```python
csv = pd.DataFrame(topics_data)
export_csv = csv.to_csv (r'~/Downloads/topics_2.csv', index = None, header=True)
print (export_csv)
```

    None
    


```python
topics_data_recover = pd.read_csv (r'~/Downloads/topics_2.csv')
```


```python
reddit_more=['rugbyunion','hellointernet','ireland']

topics_data_more=pd.DataFrame([])
a=0
for a in range(len(reddit_more)-1):
    print(a)
    target=reddit_more[a]
    subreddit = reddit.subreddit(target)
    more_subreddit = subreddit.hot()
    more_subreddit = subreddit.hot(limit=500)
    topics_dict = { "title":[], \
                "score":[], \
                "id":[], "url":[], \
                "comms_num": [], \
                "created": [], \
                "body":[]}
    for submission in more_subreddit:
        topics_dict["title"].append(submission.title)
        topics_dict["score"].append(submission.score)
        topics_dict["id"].append(submission.id)
        topics_dict["url"].append(submission.url)
        topics_dict["comms_num"].append(submission.num_comments)
        topics_dict["created"].append(submission.created)
        topics_dict["body"].append(submission.selftext)
    interim=pd.DataFrame(topics_dict)
    interim['subreddit']=interim['title']
    b=0
    for b in range(len(interim['subreddit'])):
        interim['subreddit'][b]=target
        b=+1
    topics_data_more = topics_data_more.append(interim)
    
    a=+1
topics_data_more=topics_data_more.reset_index(drop=True)
csv = pd.DataFrame(topics_data_more)
export_csv = csv.to_csv (r'~/Downloads/topics_1_more.csv', index = None, header=True)
print (export_csv)

topics_data_more['comments']=topics_data_more['title']
start=time.time()
i=0
i_int=i-1
while i <= len(topics_data_more['id']):
    if i>0 and i%100==0:
        safe_more=topics_data_more
        end=time.time()
        now=end-start
        per_it=now/(i-i_int)
        remaining=len(topics_data_more['id'])-i
        time_remaining = (remaining *per_it)/3600
        print('Done:',i,'Time per post:',per_it, 'Estimated Time Remaining:',time_remaining,'hours')
        csv = pd.DataFrame(topics_data_more)
        export_csv = csv.to_csv (r'~/Downloads/topics_safety_more.csv', index = None, header=True)
        print (export_csv)
    post=topics_data_more['id'][i]
    submission = reddit.submission(id=post)
    submission.comments.replace_more(limit=0)
    txt=""
    for comment in submission.comments.list():
        txt = txt+comment.body
    topics_data_more['comments'][i]=txt
#     txt_2=""
#     for top_level_comment in submission.comments.list():
#         txt_2 = txt_2+top_level_comment.body
#     topics_data['top_comments'][i]=txt_2
    i=i+1
csv = pd.DataFrame(topics_data_more)
export_csv = csv.to_csv (r'~/Downloads/topics_more.csv', index = None, header=True)
print (export_csv)
```

    0
    

    C:\Users\kevin.sweeney\Anaconda3\lib\site-packages\ipykernel_launcher.py:29: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
    

    1
    None
    

    C:\Users\kevin.sweeney\Anaconda3\lib\site-packages\ipykernel_launcher.py:61: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
    

    Done: 100 Time per post: 1.1916176706257433 Estimated Time Remaining: 0.2979044176564358 hours
    None
    Done: 200 Time per post: 1.1608874204740003 Estimated Time Remaining: 0.2579749823275556 hours
    None
    Done: 300 Time per post: 1.0999872819133771 Estimated Time Remaining: 0.21388641592760113 hours
    None
    Done: 400 Time per post: 1.10960228960413 Estimated Time Remaining: 0.18493371493402166 hours
    None
    Done: 500 Time per post: 1.202910210082155 Estimated Time Remaining: 0.1670708625114104 hours
    None
    Done: 600 Time per post: 1.219527166020652 Estimated Time Remaining: 0.1355030184467391 hours
    None
    Done: 700 Time per post: 1.2165141666836814 Estimated Time Remaining: 0.10137618055697345 hours
    None
    Done: 800 Time per post: 1.1830332710799505 Estimated Time Remaining: 0.0657240706155528 hours
    None
    Done: 900 Time per post: 1.1591902684159867 Estimated Time Remaining: 0.03219972967822185 hours
    None
    Done: 1000 Time per post: 1.1877438955373698 Estimated Time Remaining: 0.0 hours
    None
    


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-37-73dfcdde2789> in <module>
         53         export_csv = csv.to_csv (r'~/Downloads/topics_safety_more.csv', index = None, header=True)
         54         print (export_csv)
    ---> 55     post=topics_data_more['id'][i]
         56     submission = reddit.submission(id=post)
         57     submission.comments.replace_more(limit=0)
    

    ~\Anaconda3\lib\site-packages\pandas\core\series.py in __getitem__(self, key)
       1069         key = com.apply_if_callable(key, self)
       1070         try:
    -> 1071             result = self.index.get_value(self, key)
       1072 
       1073             if not is_scalar(result):
    

    ~\Anaconda3\lib\site-packages\pandas\core\indexes\base.py in get_value(self, series, key)
       4728         k = self._convert_scalar_indexer(k, kind="getitem")
       4729         try:
    -> 4730             return self._engine.get_value(s, k, tz=getattr(series.dtype, "tz", None))
       4731         except KeyError as e1:
       4732             if len(self) > 0 and (self.holds_integer() or self.is_boolean()):
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()
    

    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()
    

    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()
    

    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()
    

    KeyError: 1000



```python
csv = pd.DataFrame(topics_data_more)
export_csv = csv.to_csv (r'~/Downloads/topics_more.csv', index = None, header=True)
print (export_csv)
```

    None
    


```python

```
