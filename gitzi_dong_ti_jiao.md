# Git自动提交

因为有很多项目都需要push,但我在打完代码老忘记.所以写了个原始版本的crontab

具体 `crontab -e`

```shell
0 18 * * * cd /Users/maizhikun/Learning/apache_sites/Journey/   && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/Learning/apache_sites/Journey_Front/   && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/Learning/apache_sites/JavaScript_Utils/   && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/Project/Utils  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/Code_Utils  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/SQL  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/Study_React/  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/mzkmzk-web_source_study/  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/Laravel_Source_Read/  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/Study_jQuery_DataTables/  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/Read/  && git push && git add .  &&  git commit -m   "commit " && git push
0 18 * * * cd /Users/maizhikun/GitBook/Library/mzkmzk/WEB_Accumulate/  && git push && git add .  &&  git commit -m   "commit " && git push
```