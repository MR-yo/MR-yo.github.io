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

---

#### 2018.3.21 新增
 
昨天阿里一面的时候与面试官有关 UITableView 的缓存池复用有了不同的意见，于是我去看了一下 UITableView 的源码<br>
[https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UITableView.m](https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UITableView.m)

```
@implementation UITableView {
    BOOL _needsReload;
    NSIndexPath *_selectedRow;
    NSIndexPath *_highlightedRow;
    NSMutableDictionary *_cachedCells;  
    NSMutableSet *_reusableCells;      
    NSMutableArray *_sections;
    
    ...
}
```

可以看出， UITableView 有两种缓存方式 `_cachedCells` 和 `_reusableCells`<br>

1. `_cachedCells`

```
- (UITableViewCell *)cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // this is allowed to return nil if the cell isn't visible and is not restricted to only returning visible cells
    // so this simple call should be good enough.
    return [_cachedCells objectForKey:indexPath]; // 这里按 indexPath 找出 cell
}
```

```
 for (NSInteger row=0; row<numberOfRows; row++) {
    NSIndexPath *indexPath = [NSIndexPath indexPathForRow:row inSection:section];
    CGRect rowRect = [self rectForRowAtIndexPath:indexPath];
    if (CGRectIntersectsRect(rowRect,visibleBounds) && rowRect.size.height > 0) {
        UITableViewCell *cell = [availableCells objectForKey:indexPath] ?: [self.dataSource tableView:self cellForRowAtIndexPath:indexPath];
        if (cell) {
            [_cachedCells setObject:cell forKey:indexPath]; // 这里按 indexPath 添加 cell
            [availableCells removeObjectForKey:indexPath];
            cell.highlighted = [_highlightedRow isEqual:indexPath];
            cell.selected = [_selectedRow isEqual:indexPath];
            cell.frame = rowRect;
            cell.backgroundColor = self.backgroundColor;
            [cell _setSeparatorStyle:_separatorStyle color:_separatorColor];
            [self addSubview:cell];
        }
    }
}
```
查阅它的相关方法就可以看出，`_cachedCells`用于缓存`indexPath`相关的 cell

2. `_reusableCells`

```
    // remove old cells, but save off any that might be reusable
    for (UITableViewCell *cell in [availableCells allValues]) {
        if (cell.reuseIdentifier) {
            [_reusableCells addObject:cell];  // 这里只要 cell.reuseIdentifier 不为空就加入
        } else {
            [cell removeFromSuperview];
        }
    }
```

```
- (UITableViewCell *)dequeueReusableCellWithIdentifier:(NSString *)identifier
{
    for (UITableViewCell *cell in _reusableCells) {
        if ([cell.reuseIdentifier isEqualToString:identifier]) { // 这里只要 cell.reuseIdentifier 相等就取出
            UITableViewCell *strongCell = cell;
            
            // the above strongCell reference seems totally unnecessary, but without it ARC apparently
            // ends up releasing the cell when it's removed on this line even though we're referencing it
            // later in this method by way of the cell variable. I do not like this.
            [_reusableCells removeObject:cell];

            [strongCell prepareForReuse];
            return strongCell;
        }
    }
    
    return nil;
}
```

很明显，加入`_reusableCells`的是带有不为空 `identifier` 的`UITableViewCell`，而取出 `cell` 时却只判断了 `identifier` 相等就取出，虽然 `_reusableCells` 用的是`NSMutableSet`，但却不能保证 `identifier` 一样时，取出的 cell 是不是真正想要的 cell。通过使用唯一的`identifier`可以进行有效的重用，而不用再次创建 cell。

其实个人不是很理解为什么这里的`_reusableCells` 要用`NSMutableSet` 而不跟` _cachedCells` 一样，采用 `NSMutableDictionary`.很多缓存策略，包括其自身的` _cachedCells`都采用的是键值对结构。 期待大佬解惑。
