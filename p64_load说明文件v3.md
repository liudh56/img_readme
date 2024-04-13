# 数据文件

```fortran
1.0  2.0
20  20  10  5
1
27 32 30 ! 荷载
20.0  15.0  0.0  20.0  1.0e5  0.3
0.0001 500
6
1.2  1.3  1.4  1.5  1.55 1.6
```

![模型](https://pic.imgdb.cn/item/6619e7df68eb935713ab513c.png)    

其中,第四行表示荷载的加荷位置和大小（线密度）     

```fortran
27 ! 表示起点在第27列（从左至右）
32 ! 表示终点在第32列（从左至右）
30 ! 表示荷载为30kN/m
```

对应下图中的红色位置

![加载位置](https://pic.imgdb.cn/item/6619e7bc68eb935713ab1a65.png)

# 源代码

本程序单元格序号是从上至下，从左至右依次排序,如下例

```fortran
    1 6 
    2 7  11
    3 8  12 15
    4 9  13 16 18 20
    5 10 14 17 19 21
```

在叠加荷载时，因为所有单元格的g都是按照3 2 1 4 6 7 8 5 的顺序保存，因此我们仅需找到对应的单元格号以及146号节点的相对位置

146号节点的相对位置为g(6), g(8), g(10),下文有介绍

接下来只需找到单元格号即可。

## 计算单元号

```fortran
if (tree_load/=0) then !荷载不为零再进行运算
  allocate(tree_nodes(nx1+nx2+ny1*ngrad))
  
  tree_nodes=0
   ! tree_nodes(nx1+nx2+ny1*ngrad)是储存表面单元格的单元号
   
  itree = 2 !用于计数，记录需加荷的单元格数量
  tree_nodes(1)=1 
  
  ! 顶面和斜面前ngrad行
  do i=2,nx1+ngrad 
     tree_nodes(itree) = (i-1)*nye+1
     itree = itree + 1
  end do
  
  ! 斜面ngrad行以后
  do i=1,ny1-1
    do j=1,ngrad
       if (j==1) then
          tree_nodes(itree) = tree_nodes(itree-1)+nye-i+1
       else
          tree_nodes(itree) = tree_nodes(itree-1)+nye-i
       end if
       itree = itree + 1
    end do
  end do
  
  ! 底面第一列
  tree_nodes(itree) = tree_nodes(itree-1)+ny2+1
  itree = itree + 1
  
  ! 底面以后
  do i=2,nx2
     tree_nodes(itree) = tree_nodes(itree-1)+ny2
     itree = itree + 1
  end do
  
  tree_nodes_= tree_nodes(nel_left:nel_right) ! 储存实际需要加荷的单元格的序号
```

<br/>

## 分配荷载到节点

```fortran
  ALLOCATE(weight_load(8),tree(ndof))
  !荷载在节点上的分布权重
  weight_load = (/0.166667,0.0,0.0,0.666667,0.0,0.166667,0.0,0.0/)
  
  !计算各节点荷载大小
  tree=zero
  tree(6)=tree_load*weight_load(1)
  tree(8)=tree_load*weight_load(4)
  tree(10)=tree_load*weight_load(6)
  
  ! 此处注意区分(见下图)
  ! g数组按3x 3y 2x 2y 1x 1y 4x 4y 6x 6y 7x 7y 8x 8y 5x 5y的顺序保存
  ! weight_load按1 2 3 4 5 6 7 8来保存
  ! 则1 4 6号节点的y方向对应g(6),g(8),g(10)
  
end if
```

![单元节点](https://pic.imgdb.cn/item/6619e75e68eb935713aadeb2.png)

<br/>

***

## 叠加荷载

```fortran
   itree=1 ! 作tree_nodes_数组的索引，将序号依次读出
   
   elements_2: DO iel=1,nels
   
      ...
      
      gravlo(g)=gravlo(g)-eld*prop(4,etype(iel))

      if (tree_load.ne.0.0) then !荷载不为零时计算
      
         if ((iel==tree_nodes_(itree)).and.(itree.le.size(tree_nodes_))) then !判断是否为需加荷的单元
         
            gravlo(g) = gravlo(g) - tree !根据自由度编号加到指定位置
          
            itree=itree+1
         end if
      end if

   END DO elements_2
```
