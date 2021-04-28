# char 与 unsigned char

在Sunday中使用char时，发现数据跑出来不对，通过单步调试发现，跳步的大小会出错。

```
pszText[nTextpos + m_nKeyLen] < 0
m_nSunday[pszText[nTextPos + m_nKeyLen]]会有问题
```


​	通过将char 改成unsigned char 后解决这个问题。char能表示的数据范围是-128~127，而unsigned char表示的范围是0~255.