[toc]

## 指定使用某个makefiles

```bash
make -f make.Linux
make --file make.Linux
```



## 检查makefile

`make -n`或者`make --just-print`只显示命令，不执行命令，可以用来调试makefile