# 心得分享

這是我文章寫作的地方，有些已經寫好可以發表，有些還在撰寫中，所以有了以下規範。

## 發表規範

三個branch
- master
- writing
- ready

平時只有`master`與`writing`這兩個branch

要發表時使用`git checkout -b ready`生成新的`ready` branch，刪除不想發表的內容後切換到`master`並使用`git merge ready`

完成後再用`git branch -d ready`來刪除`ready` branch
