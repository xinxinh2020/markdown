[TOC]



# Xarray

```shell
# import xarray as xr

# 创建一个DataArray对象
data = xr.DataArray(np.random.randn(2,3), dims=('x','y'), coords={'x':[10,20]})
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


# return : Dataset
xarray.open_dataset(filename)

# 将数据集转成zarr格式
# store : (MutableMapping, str or Path, optional) – Store or path to directory in file system
Dataset.to_zarr(self, store)
```