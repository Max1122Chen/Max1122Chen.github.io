# 虚幻UI

使用Canvas作为UI的父组件是消耗比较大的，尽量减少这样的使用，替代方案是使用”border“或“SizeBox”



想让Widget可以叠层，需要使用Overlay



​	制作物品栏的一个思路是可以用“TileView”，通过配置这个就可以控制构成物品栏的格子，使用TileView可以实现动态按顺序填入物品栏的格子的效果，并非一张排布整齐的待填充的格子表。

![image-20250203191702654](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250203191702654.png)

但是貌似修改不了间距？？？

作为格子的Widget需要加上UserObjectListEntry接口才能放进这里面。