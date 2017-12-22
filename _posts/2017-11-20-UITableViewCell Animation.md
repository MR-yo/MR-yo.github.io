---
layout: post
title: UITableViewCell Animation
---

# UITableViewCell Animation

以前喜欢各种酷炫的列表动画,后来感觉太花哨了,一般一个渐变的效果就够了。
其实各种 animation 都能加在 cell 上,关键是要做的使用户看起来,用起来很舒服,而不是单纯地 show 一波。

> 1.y轴翻转 可以类推各种翻转
> 
> duration 如果根据 indexPath.row 来可以产生一种渐变的效果

```
    cell.layer.transform = CATransform3DMakeRotation(-90, 0, 1, 0);
    [UIView animateWithDuration:indexPath.row * 0.1 > 0.5 ? 0.5 : indexPath.row * 0.1
                          delay:0
                        options:UIViewAnimationOptionAllowUserInteraction
                     animations:^{
                         cell.layer.transform = CATransform3DIdentity;
                     } completion:nil];

```

> 2.淡入淡出 渐变

```
    cell.alpha = 0;
    [UIView animateWithDuration:indexPath.row * 0.1 > 0.5 ? 0.5 : indexPath.row * 0.1
                          delay:0
                        options:UIViewAnimationOptionAllowUserInteraction
                     animations:^{
                         cell.alpha = 1;
                     } completion:nil];
```

> 3.位移 渐变

```
    cell.alpha = 0.5;
    cell.frame = 初始位置
    [UIView animateWithDuration:0.3 animations:^{
        cell.frame = 最终位置;
        cell.alpha = 1.0;
    }];
```

> 4.弹簧动画

```
    cell.alpha = 0;
    cell.frame = 初始位置
    [UIView animateWithDuration:0.5
                          delay:0
         usingSpringWithDamping:1
          initialSpringVelocity:5
                        options:UIViewAnimationOptionCurveEaseInOut
                     animations:^{
                         cell.frame = 最终位置;
                         cell.alpha = 1;
                     } completion:^(BOOL finished) {
        
                     }];
```
