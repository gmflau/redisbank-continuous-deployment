# redisbank-continuous-deployment



Trigger a cloud-build (redisbank-branch)     
```
git checkout -b dev200
git branch 
vi dummy.txt
git add dummy.txt
git commit -m "dev200"
git push origin dev200
```


Trigger a cloud-build (redisbank-master) by merging changes from dev200  
```
git checkout master
git branch
git merge dev200
git push origin master
```


Trigger a cloud-build (redisbank-master) by merging changes from dev200  
```
git checkout master
git branch
git tag v2.0.1
git push origin v2.0.1
```
