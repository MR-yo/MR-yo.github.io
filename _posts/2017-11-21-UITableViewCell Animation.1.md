---
layout: post
author: syea
title: UITableView缓存池
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - iOS
    - Objective-C
---

# 关于 UITableView 缓存池

`UITableView` 是 iOS 开发中比较常用的控件，对于 cell 的重用，网上很多资料都是这样写的

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *cellId = @"hello";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
    }
    cell.textLabel.text = [NSString stringWithFormat:@"%@  --  %ld",_dataArray[indexPath.row],indexPath.row];
    return cell;
}
```

如果有自定义视图，那么每次都要加载一遍视图，加载前还得把前面加的视图清空。<br />
一开始学习的时候什么都不懂，我也这样写，后来我就想，这样写的话，缓存池的意义在哪？<br />
`UITableView` 通过 `Identifier` 应该可以直接从缓存池中拿出 cell 才对。<br />

那么就这样写

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *cellId = @"hello";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
        cell.textLabel.text = [NSString stringWithFormat:@"%@  --  %ld",_dataArray[indexPath.row],indexPath.row];
    }
    return cell;
}
```

然而，每个被放入缓存池 cell 绑定的 `Identifier` 都是同一个，取出来的时候就会混乱。<br />
那怎么设置不同的`Identifier`。<br />

1. 数据不会变的情况下，可以通过 `indexPath` 设置
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSString *cellId = [NSString stringWithFormat:@"hello-%ld-%ld",indexPath.section,indexPath.row];
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
        cell.textLabel.text = [NSString stringWithFormat:@"%@  --  %ld",_dataArray[indexPath.row],indexPath.row];
    }
    return cell;
}
```
结果不会乱序,再加载完所有的 cell 后，也不会再进行初始化 cell，都是从缓存池取。


2. 数据会变的情况
这个情况占大多数，有时候下拉刷新获取新数据就改变了 `indexPath`，导致绑定 `Identifier` 出现重复<br />
我的解决方案是  **数据源绑定唯一id**
```
- (NSString *)getIdentifier
{
    // 暂时不考虑 MD5 加密 怕太耗时
    // 1. 时间戳
    NSTimeInterval time = [[NSDate date] timeIntervalSince1970] * 1000;
    double i = time;
    NSString *timeString = [NSString stringWithFormat:@"%.f",i];
    // 2. 随机数
    NSInteger j = random() % 99;
    NSString *result = [NSString stringWithFormat:@"id%@%ld",timeString,j];
    return result;
}
```
```
    // MomentsModel 绑定一个 cellId 属性。
    _momentCellId = [self getIdentifier];
```
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    MomentsModel *model = _dataArray[indexPath.row];
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:model.momentCellId];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:model.momentCellId];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.backgroundColor = [UIColor clearColor];
        MomentsItemView *itemView = [[MomentsItemView alloc] initWithFrame:CGRectMake(0, 0, kScreenWidth, model.cellHeight) model:model];
        itemView.indexPath = indexPath;
        itemView.delegate = self;
        [cell addSubview:itemView];
    }
    return cell;
}
```
**`getIdentifier`方法可以抽象出去, 实体类的`cellId`属性也可以通过 runtime 解耦**<br />
但是这样重新获取数据源的时候，由于 id 会变，所有的 cell 还是要重新创建而不会去缓存池拿。<br />
鉴于重新获取数据源的时候，的确需要刷新界面，比如一些状态值变了，界面上也要跟着变。<br />

#### 真的没有问题了吗？

`UITableView` 会不会因为不断地往缓存池中缓存 cell 导致占用内存越来越大？<br />
所有的 cell 都缓存起来这种处理方式真的好吗？<br />

于是我在获取数据源的地方设置了不停写入数据，并在 cell 的初始化方法处打了断点。<br />
滑动 UITableView 时:
1. 占用内存并没有越来越大。
2. 在划过大概10几个 cell 的时候，cell 又进入了初始化方法。

## 结论
1. UITableView 的缓存池有上限，会自己处理。<br />
2. 通过数据源绑定唯一 id 可以真正地用上 UITableView 的缓存策略。<br />

