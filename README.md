# 心得分享

這是我文章寫作的地方，有些已經寫好可以發表，有些還在撰寫中，所以有了以下規範。

## 發表規範

#### 兩個branch
- master
- writing

#### 文章位置
- 寫作中的內容放在`articles`底下
- 要發表的內容放在`articles\posts`

#### 發表流程
1. 要發表時使用`git checkout master` 切換到 master branch
2. `git merge writing`
3. 刪除不想發表的內容
4. `git add .`
5. `git commit -am "ready to push"`
6. `git push github master`
7. 切換回`writing` branch

# ToDo
