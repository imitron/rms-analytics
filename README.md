# Кто есть кто в компании за отмену Столлмана

Компания "за отмену Столлмана", начавшаяся с публикации в Medium (https://selamjie.medium.com/remove-richard-stallman-fec6ec210794) предоставляет нам множество интересных данных. Так как подписание открытых писем за отмену (https://rms-support-letter.github.io/) и в поддержку Столлмана (https://rms-support-letter.github.io/) осуществляется на гитхабе, мы можем проанализировать некоторые характеристики обоих сторон, используя статистические данные, которые доступны через API.

Этому помогает то, что на гитхабе затруднительно редактировать данные "задним числом" без потери новых подписей.

Следующие предположения можно проверить ("X" может быть как предложением отменить Столлмана, так и выражением его поддержки):

-   Противники X чаще ассоциированы с крупными компаниями чем сторонники
-   Сторонники X чаще и больше коммитят код и этим более полезны сообществу СПО.
-   Противники X значимо реже коммитят в репозитории со свободными лицензиями.
-   Противники X предпочитают Rust (или JS), сторонники предпочитают C (или C++, Python)
-   Противники X в большей степени социально активны, у них есть аккаунты в соц. сетях, твиттере, они часто пишут.
-   Противники X не коммитят код по выходным (работают только в рабочее время, не энтузиасты)
-   Большинство противников X зарегистрированы на гитхабе менее полугода назад

Мы попытались проверить некоторые из этих предположений и приглашаем всех, кто заинтересовался, проверить остальные предположения и вносить (и проверять) любые другие.

Мы создали репозиторий (ссылка), в котором будет проходить работа (https://github.com/imitron/rms-analytics/), в нем же лежит эта статья, ее копия на хабре будет актуализироваться по мере добавления пулл-реквестов. Присоединяйтесь к исследованию!

Далее будут детали.

# Замечание о научной честности

Любые гипотезы и любые проверяемые подтверждения будут приняты и добавлены в статью. Мы не считаем возможным скрывать данные, которые противоречат нашей позиции. Все интерпретации будут также добавлены. Мы приглашаем к совместной работе сторонников обоих позиций (да, это возможно)

# О получении данных

API и документация github: https://docs.github.com/en/rest

# Компания за отмену Столлмана управляется из одного центра

Репозиторий противников Столлмана был создан Tue 23 Mar 2021 10:42:36 AM PDT, сторонников - Tue 23 Mar 2021 01:23:39 PM PDT. Видно, что репозиторий противников практически сразу начал активно набирать звезды. У репозитория сторонников был длительный период, когда звезды набирались медленно, но потом (видимо после публикации в соц сетях) процесс пошел много быстрее и количество звезд быстро обогнало противников.

[Картинка]

```bash
\#!/bin/bash

set -ue

page=1

owner_repo=$1

while true; do
    curl -s -H "Authorization: token $GITHUB_OAUTH_" \\
        -H "Accept: application/vnd.github.v3.star+json" \\
        "<https://api.github.com/repos/$owner_repo/stargazers?per_page=100&page=$page>"| \\
        jq -r .[].starred_at_ | grep . || break
    ((page++)) || true
done

echo "epoch,con" >con.stars.csv
./get-stars.sh 'rms-open-letter/rms-open-letter.github.io'|while read a; do date -d $a +%s; done|sort -n|cat -n|awk '{print $2","$1}' >>con.stars.csv

echo "epoch,pro" >pro.stars.csv
./get-stars.sh 'rms-support-letter/rms-support-letter.github.io'|while read a; do date -d $a +%s; done|sort -n|cat -n|awk '{print $2","$1}' >>pro.stars.csv

join -t, -e '' -o auto -a1 -a2 con.stars.csv pro.stars.csv >joined.stars.csv
```

При этом спустя много дней репозиторий сторонников продолжает набирать звезды, в то время как у противников процесс сильно замедлился. Из этого можно сделать предположение, что процесс раскрутки инициативы противников был заранее интенсифицирован рассылкой писем и сообщений в социальных сетях, и замедлился как только доступная аудитория была выбрана.

Инициатива сторонников по-видимому опирается на широкие массы людей без единого управляющего центра. Этим можно объяснить медленный темп набора звезд в начале и то, что звезды до сих пор добавляются - новости расходятся от свежих вовлеченных участников.
                                                                                                                          
Теперь давайте посмотрим на активность в самих репозиториях.                                                            
                                                                                                                          
На момент написания этой статьи было 1345 комиттеров противников и 5000+ коммиттеров сторонников. Скачиваем историю коммитов:

```python
\#!/usr/bin/env python

import os
import requests
import json
import sys

repo = sys.argv[1]

headers = {'Authorization': 'token {}'.format(os.environ["GITHUB_OAUTH"])}
commits = []
page = 0
while page < 300:
    page += 1
    data = requests.get('https://api.github.com/repos/{}/commits?per_page=100&page={}'.format(repo, page), headers=headers).json()
    if len(data) == 0:
        break
    commits += data

print(json.dumps(commits, indent=4))

./get-commits.py 'rms-open-letter/rms-open-letter.github.io' >con.commits.json
./get-commits.py 'rms-support-letter/rms-support-letter.github.io' >pro.commits.json
```

Посмотрим на изменение количества коммитов от времени:

```bash
jq -r .[].commit.author.date pro.commits.json|sort -u|cat -n|awk '{print $2","$1}'|sed -e 's/T/ *' -e 's/Z/*' >pro.commits.csv
jq -r .[].commit.author.date con.commits.json|sort -u|cat -n|awk '{print $2","$1}'|sed -e 's/T/ *' -e 's/Z/*' >con.commits.csv
join -t, -e '' -o auto -a1 -a2 con.commits.csv pro.commits.csv >joined.commits.csv
```
[Картинка]

Видно, что репозиторий сторонников гораздо активнее. Коммитов в репозиторий противников за последнее время практически не было. Репозиторий сторонников продолжает обновляться. 

Посмотрим на распределение коммитов по дням недели.

```bash
jq -r .[].commit.author.date con.commits.json |./weekday-from-date.py >con.rms_commits.csv
jq -r .[].commit.author.date pro.commits.json |./weekday-from-date.py >pro.rms_commits.csv
join -t, con.rms_commits.csv pro.rms_commits.csv >joined.rms_commits.csv
```

[Картинка]

Активность сторонников значительно менее вариативная. Коммиты совершаются во все дни недели. 

Aктивность противников Столлмана сильно снижается на выходных, зато в среду мы видим пик. Это можно объяснить тем, что во многих компаниях среда это no meeting day.

Скачиваем индивидуальные данные для каждого юзера, а также его последние 100 действий:

```bash
jq -r .[].author.login con.commits.json|sort -u >con.logins
jq -r .[].author.login pro.commits.json|sort -u >pro.logins

\#!/bin/bash

set -ue

script_dir=$(dirname $(realpath $0))

get_data() {
    local data_dir=$script_dir/$1 userdata events
    for x in $(cat $1.logins); do
        userdata=$data_dir/$x.userdata
        [ -r $userdata ] && continue
        curl -s -H "Authorization: token $GITHUB_OAUTH" "<https://api.github.com/users/$x>" >$userdata
        sleep 1
        events=$data_dir/$x.events
        [ -r $events ] && continue
        curl -s -H "Authorization: token $GITHUB_OAUTH" "<https://api.github.com/users/$x/events?per_page=100>" >$events
        sleep 1
    done
}

get_data $1

./get-user-events-data.sh con
./get-user-events-data.sh pro
```

Пример данных юзера, выгруженных из гитхаба:

```json
{
  "login": "zyxw59",
  "id": 3157093,
  "node_id": "MDQ6VXNlcjMxNTcwOTM=",
  "avatar_url": "https://avatars.githubusercontent.com/u/3157093?v=4",
  "gravatar_id": "",
  "url": "https://api.github.com/users/zyxw59",
  "html_url": "https://github.com/zyxw59",
  "followers_url": "https://api.github.com/users/zyxw59/followers",
  "following_url": "https://api.github.com/users/zyxw59/following{/other_user}",
  "gists_url": "https://api.github.com/users/zyxw59/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/zyxw59/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/zyxw59/subscriptions",
  "organizations_url": "https://api.github.com/users/zyxw59/orgs",
  "repos_url": "https://api.github.com/users/zyxw59/repos",
  "events_url": "https://api.github.com/users/zyxw59/events{/privacy}",
  "received_events_url": "https://api.github.com/users/zyxw59/received_events",
  "type": "User",
  "site_admin": false,
  "name": "Emily Crandall Fleischman",
  "company": "Commure",
  "blog": "",
  "location": null,
  "email": "emilycf@mit.edu",
  "hireable": null,
  "bio": null,
  "twitter_username": null,
  "public_repos": 24,
  "public_gists": 0,
  "followers": 2,
  "following": 12,
  "created_at": "2012-12-31T05:33:30Z",
  "updated_at": "2021-03-14T01:53:51Z"
}
```

В таблице ниже приводится процент пользователей, у которых заполнены поля twitter_username>, company, bio и blog:

<table border="2" cellspacing="0" cellpadding="6">
<tbody>
<tr>
<td>поле</td>
<td>противник</td>
<td>сторонник</td>
</tr>
<tr>
<td>twitter_username</td>
<td>31%</td>
<td>8%</td>
</tr>
<tr>
<td>company</td>
<td>48%</td>
<td>20%</td>
</tr>
<tr>
<td>bio</td>
<td>53%</td>
<td>31%</td>
</tr>
<tr>
<td>blog</td>
<td>63%</td>
<td>31%</td>
</tr>
</tbody>
</table>

Противники значимо более социально активны. Можно предположить что участие в социальных акциях, таких как подписание письма для сторонников Столлмана является более непривычным актом чем для противников.

Посмотрим на поля public_repos, public_gists, followers и following: 

<table>
  <tbody> 
    <tr> <td>поле</td> <td >противник</td> <td>противник</td> <td >сторонник</td><td>противник</td></tr>
    <tr> <td></td> <td>average</td> <td>median</td> <td>average</td> <td>median</td> </tr>
    <tr> <td>public_repos</td> <td>62</td> <td>34</td> <td>21</td> <td>9</td> </tr>
    <tr> <td>public_gists</td> <td>18</td> <td>4</td> <td>4</td> <td>0</td> </tr>
    <tr> <td>followers</td> <td>105</td> <td>23</td> <td>16</td> <td>2</td> </tr>
    <tr> <td>following</td> <td>30</td><td>8</td> <td>14</td> <td>1</td> </tr> 
    </tbody> 
</table>

Опять видно, что противники Столлмана гораздо активнее сторонников на гитхабе. Можно предположить, что это разработчики популярных проектов, деятельность которых интересна многим. Видимо именно это объясняет столь высокое значение поля followers. Также обращает на себя внимание, что у противников соотношение followers / following больше 3, в то время как у сторонников оно составляет 1.1. Давайте воспользуемся полем events_url, чтобы скачать историю действий юзеров.                              
                                                                                                                                                        
Теперь давайти посмотрим на действия юзеров. Данных скачано много и анализировать их можно множеством способов. Можно проверить активность юзеров по дням недели, чтобы проверить как эти данные коррелируют с активностью, специфичной для репозиториев "за" и "против" Столлмана.

```python
\#!/usr/bin/env python                                                                                                                                  
                                                                                                                                                        
import datetime                                                                                                                                         
import sys                                                                                                                                              
                                                                                                                                                        
out = [0] \* 7                                                                                                                                          
total = 0                                                                                                                                               
                                                                                                                                                        
for line in sys.stdin.readlines():                                                                                                                      
    weekday = datetime.datetime.strptime(line.strip(), '%Y-%m-%dT%H:%M:%SZ').weekday()                                                                  
    out[weekday] += 1                                                                                                                                   
    total += 1                                                                                                                                          
                                                                                                                                                        
for day, count in enumerate(out):                                                                                                                       
    print("{},{}".format(day, count / total))                                                                                                           
                                                                                                                                                        
jq -r .[].created<sub>at</sub> con/\*.events|./weekday-from-date.py >con.event<sub>day.normalized.csv</sub>                                             
jq -r .[].created<sub>at</sub> pro/\*.events|./weekday-from-date.py >pro.event<sub>day.normalized.csv</sub>                                             
join -t, con.event<sub>day.normalized.csv</sub> pro.event<sub>day.normalized.csv</sub>  
```

[Картинка]

Видно, что тренд сохранился: активность противников резко снижается по выходным. Можно предполагать, что они используют гитхаб на работе и, возможно, работают над open source проектами за зарплату. Если это предположение верно, их мнение может быть обусловлено отбором, который проводят компании, нанимающие программистов для работы над open source проектами.                                                                                                
