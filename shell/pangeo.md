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
data.loc[10]
data.isel(x=0) # 使用维的名称和一个整型的标签
data.sel(x=10) # 使用维的名称和一个坐标轴的标签

# Load and decode a dataset from a Zarr store	
# store (MutableMapping or str) – A MutableMapping where a Zarr Group has been stored or a path to a directory in file system where a Zarr DirectoryStore has been stored
# return : Dataset
xarray.open_zarr(store) 

# return : Dataset
xarray.open_dataset(filename)

# 将数据集转成zarr格式
# store : (MutableMapping, str or Path, optional) – Store or path to directory in file system
Dataset.to_zarr(self, store)
```

