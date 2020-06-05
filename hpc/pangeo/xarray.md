[TOC]

# 概览

Xarray不仅跟踪数组上的标签—它还使用它们来提供强大而简洁的接口。例如:

- 按名称对维度应用操作:x.sum('time')
- 根据标签(或逻辑位置)而不是整数位置选择值:x.loc(“2014-01-01”)或x.sel(time= ' 2014-01-01 ')
- 数学运算(例如，x - y)基于维度名称向量化多个维度(数组广播)，而不是形状

xarray数据结构的n维特性使其适合处理多维科学数据。它使用维度名代替轴标签(dim='time'代替axis=0)



## 核心数据结构

xarray有两个核心数据结构，它们构建并扩展了NumPy和panda的核心优势。这两种数据结构都是n维的:

- DataArray：是我们实现的带标签的n维数组，它是pandas.Series的N-D泛化
- Dataset：是一个多维的、内存中的数组数据库。它是一个类似于字典的DataArray对象容器，它沿着任意数量的共享维度排列，在xarray中它的作用与pandas.DataFrame类似

与普通字典相比，数据集的强大之处在于，除了按名称提取数组外，还可以同时跨所有数组选择或组合一个维度上的数据。与DataFrame一样，数据集促进了对异构数据的数组操作——不同之处在于数据集中的数组不仅可以有不同的数据类型，还可以有不同的维数。

这个数据模型借鉴了netCDF文件格式，它还为xarray提供了一种自然的、可移植的序列化格式。NetCDF在地球科学中非常流行，而且已有许多用于在许多编程语言(包括Python)中读写NetCDF的库。

xarray与许多处理netCDF数据的工具不同，它提供了用于内存分析的数据结构，这些数据结构既可以利用标签，也可以保存标签。您只需执行一次添加元数据的繁琐工作，而不是每次保存文件时都添加元数据。



## 目标和期望

Xarray为Python用于数值计算的SciPy生态系统提供了与域无关的数据结构和标记多维数组的工具。特别是，xarray构建并集成了NumPy和panda:

- 我们面向用户的接口旨在提供NumPy/panda中所提供的接口的更明确的版本
- 与更广泛的生态系统兼容是一个主要目标:应该很容易地获取数据
- 我们努力将重点放在与标记数据相关的功能和接口上，并利用其他Python库来处理其他所有事情，例如，NumPy/panda用于快速数组/索引(xarray本身不包含已编译的代码)、Dask用于并行计算、matplotlib用于绘图等

Xarray是一个协作和社区驱动的项目，完全依靠志愿者的努力来运行。我们的目标受众是需要Python中的n维标记数组的任何人。最初，开发是由物理科学家(特别是已经了解并热爱netCDF的地球科学家)的数据分析需求驱动的，但现在它已经成为一个更广泛的有用工具，并且仍在积极开发中。



# FAQ

## 为什么不用pandas就可以了

pandas在处理低维（小于或等于2维）数组时很棒。

panda过去一直支持n维面板，但在0.20版中弃用了它们，转而支持Xarray数据结构。现在，两边都有内置的方法来在panda和Xarray之间进行转换，从而可以进行更集中的开发工作。Xarray对象有一个更丰富的维度模型-如果你使用面板:

- 您需要为每个维度创建一个新的工厂类型
- 你无法在不同维度的ndpanel之间进行数学运算
- NDPanel中的每个维度都有一个名称(例如，' label '、' items '、' major_axis '等)，但是维度名称指的是顺序，而不是它们的含义。您不能指定要沿“时间”轴应用的操作（意思应该是只能指定第几个轴）
- 您通常必须手动转换panda数组的集合(系列、DataFrames等)以获得相同数量的维度。相反，这种数据结构非常自然地适合于xarray数据集

## xarray的数据结构和pandas有什么不同

xarray的DataArray和panda中标记的数组的主要区别在于维度可以有名称(例如，“时间”、“纬度”、“经度”)，名称比轴号更容易跟踪，而且xarray使用维度名称进行索引、聚合和广播。Not only can you write `x.sel(time='2000-01-01')` and `x.mean(dim='time')`, but operations like `x - x.mean(dim='time')` always work, no matter the order of the “time” dimension. You never need to reshape arrays (e.g., with `np.newaxis`) to align them for arithmetic operations in xarray.



## xarray的哪些部分可以认为是公共的API

API references中的函数和方法可以认为是对外暴露的API，其它的函数和方法则是实现上的细节



# 快速概览

如果不同的变量都有x维，那在所有变量中的x维都必须是一样的。



# 用户指导

## 数据结构

### Dataset

它是一个类似字典的标签数组(DataArray对象)容器，具有对齐的维度。它被设计成netCDF文件格式的数据模型在内存中的表示形式。它有四个重要的属性：

- dims : 一个从维度的名称到每个维度的固定长度的映射，如{'x': 6, 'y': 6, 'time': 8}
- data_vars : 与变量相对应的类似于字典的DataArray容器
- coords : 

### Coordonates

Coords是Dataset和DataArray的辅助变量。和其他属性不一样，coords在转换对象的时候会被翻译和持久化。有两种coords:

- **dimension coordinates:** 某一个维度的坐标，它的名字和这个维度的名字相同（打印的时候前面带有*号），它们用于基于标签的索引和对齐，类似于数字的下标或者是pandas中的index,实际上，它内部使用了pandas的index来存储它的值
- **non-dimension coordinates：**是包含坐标数据，但不是维度坐标的变量。它可以是多维的，它的名字和它的维度没有什么关系。非维度坐标可以用于索引或绘制。xarray不会直接利用这些值



## 索引和选择数据

Dataset不支持位置索引，因为Dataset中维度的顺序是不明确的。



## 与Dask结合

Dask把数组分成小片，称为chunk，每个chunk足够小，可以加载到内存中。和numpy不一样，操作在dask中是lazy的。操作映射成一系列块上的任务，排成队列，在实际要求计算值之前不执行任何计算。当要进行计算时，数据才加载进内存并进行计算。

几乎所有现有的xarray方法(包括用于索引、计算、连接和分组操作的方法)都被扩展为自动使用Dask数组。当您在一个xarray数据结构中将数据作为一个Dask数组加载时，几乎所有的xarray操作都会将其作为一个Dask数组;当这是不可能的时候，它们将引发一个异常，而不是意外地将数据加载到内存中。

如果要使用一个xarray不支持的函数，可以将dask数组从xarray中提取出来（.data）直接使用dask数组

# Shell命令

```shell
# import xarray as xr

# 创建一个DataArray对象
da = xr.DataArray(np.random.randn(2,3), dims=('x','y'), coords={'x':[10,20]})
data.values # 返回一个numpy数组
data.dims # 返回维度信息
data.coords # 坐标轴信息
data.attrs # 存放任意自定义属性，cn格式常用的属性：units、long_name
data[0, :] # 使用数字下标索引
data.loc[10,...] # 使用标签索引（第1维....第2维....）
data.isel(x=0) # 使用维的名称和一个整型的标签  无需关注维的顺序
data.sel(x=10,[method=['nearest|pad|backfill']]) # 使用维的名称和一个坐标轴的标签
data.plot() # 绘制图表
ds = da.to_dataset(name="foo") # 将DataArray转换成Dataset
da.where(da.y < 2[, drop=True]) # 条件选择，drop=True没有匹配到的会被删除，而不是nan
da.isin([2, 4]) # 

# Load and decode a dataset from a Zarr store	
# store (MutableMapping or str) – A MutableMapping where a Zarr Group has been stored or a path to a directory in file system where a Zarr DirectoryStore has been stored
# return : Dataset
xarray.open_zarr(store) 

############ Dataset ###########
# 将Dataset对象保存到nc文件中
ds.to_netcdf('example.nc')
# 打开一个nc文件
xr.open_dataset('example.nc')
# 删除Dataset的变量
ds.drop_vars('temperature')
# 删掉一个维度，所有使用到该维度的变量都会被删除
ds.drop_dims('time')
# 删掉选中数据
ds.drop_sel(space=['IN', 'IL'])
# 重命名变量
ds.rename({'temperature': 'temp', 'precipitation': 'precip'})
# 返回一个新的Dataset对象，该对象增加一个新的变量temperature2
ds.assign(temperature2 = 2 * ds.temperature)
# 把维度time和变量day进行交换
ds.swap_dims({'time': 'day'})
# 将所有非维度坐标转换成变量
ds.reset_coords()
# 将变量转换成非维度坐标
ds.set_coords(['temperature', 'precipitation'])
# 将coords转换成真正的pandas.Index
ds['time'].to_index()
# 每个维度对应的索引
ds.indexes
ds.nbytes # ds的大小
rechunked = ds.chunk({'latitude': 100, 'longitude': 100}) # 把整个数据集切分成chunk
rechunked.chunks # 查看数据集的chunk

# 立刻将数据加载到内存中 
ds.load()
# 将数据加载到分布式内存中，并保存dask数组的格式
ds.persist()


# return : Dataset
xarray.open_dataset(filename)

# 将数据集转成zarr格式
# store : (MutableMapping, str or Path, optional) – Store or path to directory in file system
Dataset.to_zarr(self, store)
```

