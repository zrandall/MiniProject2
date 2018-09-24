# MiniProject2: Discover a list of projects on sourceforge.net and gitlab.com


These two forges presente two different types of challeges to discovery. 

SourceForge actively prevents discovery. SourceForge over ten years ago was he largest forge 
but as it started losing its market share to other forges it started blocking project discovery. 

GitLab, on the other hand, has an error-prone API that is highly unreliable. 



Discover at least 50 projects on each of the forges that have 
names starting with the letter (case insensitive) in font of your name. 
Please provide ipython notebook used to do the discovery.

You are free to use any method, including a list compiled by someone else, search on google search engine, etc. 
but you do need to verify that the discovered projects do currently exist on these 
forges by retrieving the url of the version control repository used by the project. 

Please do discovery using the googlecloud VM to avoid accidentally 
triggereing blocking to others. 

```
a;3PIV;pprovins;Provins IV, Preston
b;BrettBass13;bbass11;Bass, Brett Czech
c;CipherR9;gyj992;Johnson, Rojae Antonio
d;Colsarcol;cmawhinn;Mawhinney, Colin Joseph
e;EvanEzell;eezell3;Ezell, Evan Collin
f;MikeynJerry;jdunca51;Duncan, Jerry
g;Tasmia;trahman4;Rahman, Tasmia
h;awilki13;awilki13;Wilkinson, Alex Webb
i;bryanpacep1;jpace7;Pace, Jonathan Bryan
j;caiwjohn;cjohn3;John, Cai William
k;cflemmon;cflemmon;Flemmons, Cole
l;dbarry9;dbarry;Barry, Daniel Patrick
m;desai07;adesai6;Desai, Avie
n;gjones1911;gjones2;Jones, Gerald Leon
o;herronej;eherron5;Herron, Emily Joyce
p;hossain-rayhan;rhossai2;Hossain, Rayhan
q;jdong6;jdong6;Dong, Jeffrey Jing
r;jyu25utk;jyu25;Yu, Jinxiao
s;mkramer6;mkramer6;Kramer, Matthew S
t;mmahbub;mmahbub;Mahbub, Maria
u;nmansou4;nmansou4;Mansour, Nasib
v;nschwerz;nschwerz;Schwerzler, Nicolas Winfield William
w;rdabbs42;rdabbs1;Dabbs, Rosemary
x;saramsv;mousavi;Mousavicheshmehkaboodi, Sara
y;spaulsteinberg;ssteinb2;Steinberg, Samuel Paul
z;zol0;akarnauc;Karnauch, Andrey
a;zrandall;zrandall;Randall, Zachary Adams
b;lpassarella;lpassare;Passarella, Linsey Sara
c;tgoedecke;pgoedec1;Goedecke, Trish
d;ray830305;hchang13;Chang, Hsun Jui
e;ssravali;ssadhu2;Sadhu, Sri Ravali
f;diadoo;jpovlin;Povlin, John P
g;mander59;mander59;Anderson, Matt Mcguffee
h;iway1;iway1;Way, Isaac Caldwell
```

## GitLab discovery  
GitLab provides [APIs](https://docs.gitlab.com/ee/api/) to retrieve project url.  
Here is a sample code for collecting projects url:
```
import sys
import re
import pymongo
import json
import time
import datetime
import requests

dbname = sys.argv[1]
collname = sys.argv[2]
# beginning page index
begin = sys.argv[3]
client = pymongo.MongoClient(host=YOURHOST)

db = client[dbname]
coll = db[collname]


beginurl = "https://gitlab.com/api/v4/projects?archived=false&membership=false&order_by=created_at&owned=false&page=" + begin + \
    "&per_page=99&simple=false&sort=desc&starred=false&statistics=false&with_custom_attributes=false&with_issues_enabled=false&with_merge_requests_enabled=false"


gleft = 0

header = {'per_page': 99}

# check remaining query chances for rate-limit restriction
def wait(left):
    global header
    while (left < 20):
        l = requests.get('https://gitlab.com/api/v4/projects', headers=header)
        if (l.ok):
            left = int(l.headers.get('RateLimit-Remaining'))
        time .sleep(60)
    return left

# send queries and extract urls 
def get(url, coll):

    global gleft
    global header
    global bginnum
    gleft = wait(gleft)
    values = []
    size = 0

    try:
        r = requests .get(url, headers=header)
        time .sleep(0.5)
        # got blocked
        if r.status_code == 403:
            return "got blocked", str(bginnum)
        if (r.ok):

            gleft = int(r.headers.get('RateLimit-Remaining'))
            lll = r.headers.get('Link')
            t = r.text.encode('utf-8')
            array = json.loads(t)
            
            for el in array:
                coll.insert(el)
 
            #next page
            while ('; rel="next"' in lll):
                gleft = int(r.headers.get('RateLimit-Remaining'))
                gleft = wait(gleft)
                # extract next page url
                ll = lll.replace(';', ',').split(',')
                url = ll[ll.index(' rel="next"') -
                         1].replace('<', '').replace('>', '').lstrip()
             
                try:
                    r = requests .get(url, headers=header)
                    if r.status_code == 403:
                        return "got blocked", str(bginnum)
                    if (r.ok):
                        lll = r.headers.get('Link')
                        t = r.text.encode('utf-8')
                        array1 = json.loads(t)
                        for el in array1:
                            coll.insert(el)
                    else:
                        sys.stderr.write("url can not found:\n" + url + '\n')
                        return 
                except requests.exceptions.ConnectionError:
                    sys.stderr.write('could not get ' + url + '\n')

        else:
            sys.stderr.write("url can not found:\n" + url + '\n')
            return

    except requests.exceptions.ConnectionError:
        sys.stderr.write('could not get ' + url + '\n')
    except Exception as e:
        sys.stderr.write(url + ';' + str(e) + '\n')
        
#start retrieving        
get(beginurl,coll)
```
Note that the parameters in the sample code are not in a perfect setting. Please feel free to tune them. This sample code is not robust enough to deal with various returned errors from query. You might need to investigate errors encountered individually. 
