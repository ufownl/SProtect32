# VC++使用说明

请仔耐心细阅读SDK的说明, 这对于你今后保护自己的软件会有极大的帮助.

注意:
WEB端 网络授权系统, 一共支持40个水印使用, 请妥善安排你的加密方案, 如需更多的加密水印支持请更换 WEB端全功能版本.
WEB端 全功能, 一共支持512个水印使用, 请妥善安排你的加密方案, 如需更多的加密水印支持请更换 PC版.

1. 请注意, 本SDK仅仅支持32位的PE文件

2. 驱动PE文件加密, 仅仅只支持代码加密SDK, 其他SDK均不可在驱动PE文件加密时使用

3. 请勿在SEH异常处理块的{ }中使用 代码加密`SP_Begin, SP_End`跟防止Dump代码加密`SP_AntiDumpCodeBegin, SP_AntiDumpCodeEnd`2种水印， 比如:
```C++
__try
{
	int *p = 0;
	*p = 3;
}
__except (EXCEPTION_EXECUTE_HANDLER)
{
	SP_Begin(1);		// 请勿这样使用代码加密的水印
	printf("error.\n");
	SP_End();
}
```
	
正确的姿势:
```C++
SP_Begin(1);

__try
{
	int *p = 0;
	*p = 3;
}
__except (EXCEPTION_EXECUTE_HANDLER)
{
	printf("error.\n");
}
	
SP_End();
```
	
4. 代码加密水印`SP_Begin, SP_End`可以包含 防止Dump代码加密水印`SP_AntiDumpCodeBegin, SP_AntiDumpCodeEnd`, 比如:
```C++
int main(void)
{
	SP_Begin(1);
	
	SP_AntiDumpCodeBegin();		// 被 SP_AntiDumpCodeBegin SP_AntiDumpCodeEnd 包含的代码会被抽取进行变异膨胀处理(轻微)然后放置在动态内存中执行[支持seh]
	printf("hello, sp1!\n");
	SP_AntiDumpCodeEnd();
	
	SP_AntiDumpCodeBegin();
	printf("hello, sp2!\n");
	SP_AntiDumpCodeEnd();
	
	SP_AntiDumpCodeBegin();
	printf("hello, sp3!\n");
	SP_AntiDumpCodeEnd();
	
	SP_End();
	
	return 0;
}
```

5. 建议每一个代码加密水印`SP_Begin, SP_End`, 都或多或少的包含3 - 4个`SP_AntiDumpCodeBegin, SP_AntiDumpCodeEnd`水印, 这样可以更好的保护你们的APP被DUMP

6. 相反, 防止Dump代码加密水印`SP_AntiDumpCodeBegin, SP_AntiDumpCodeEnd` 不能包含 代码加密水印`SP_Begin, SP_End`, 比如:
```C++
int main(void)
{
	SP_AntiDumpCodeBegin(1);
	
	SP_Begin();		// 不能在 SP_AntiDumpCodeBegin 中包含 代码加密水印[SP_Begin, SP_End]
	printf("hello, sp1!\n");
	SP_End();
	
	SP_Begin();
	printf("hello, sp2!\n");
	SP_End();
	
	SP_Begin();
	printf("hello, sp3!\n");
	SP_End();
	
	SP_AntiDumpCodeEnd();
	
	return 0;
}
```

7. 代码内存效验水印`SP_CRC`是一个可以接收返回值的水印, 但是它也有一定的限制, 使用方法如下:
```C++
int main(void)
{
	SP_Begin(1);
	
	printf("hello, sp1!\n");
	printf("hello, sp2!\n");
	printf("hello, sp3!\n");
	
	int iCrcStatus = 0;  // 仅仅支持局部变量
	SP_CRC(iCrcStatus);
	if (iCrcStatus)
	{
		printf("代码内存被修改!\n");
	}
	else
	{
		printf("代码内存正常!\n");
	}
	
	SP_End();
	
	return 0;
}
```

8. 代码内存效验水印`SP_CRC_AUTO`是一个发现异常自动结束进程的水印(如果不需要上传被修改的日志, 推举你使用这个水印), 使用方法如下:
```C++
int main(void)
{
	SP_Begin(1);
	
	printf("hello, sp1!\n");
	printf("hello, sp2!\n");
	printf("hello, sp3!\n");
	
	SP_CRC_AUTO();	// 一旦发现代码内存被修改, 则自动结束进程
	
	SP_End();
	
	return 0;
}
```

9. 反调试水印`SP_ANTIDEBUG`是一个用来检测动态调试的水印, 一旦发现被调试会返回一个非0值, 使用方法如下:
```C++
int main(void)
{
	SP_Begin(1);
	
	printf("hello, sp1!\n");
	printf("hello, sp2!\n");
	printf("hello, sp3!\n");
	
	int iAntiDebugStatus = 0;  // 仅仅支持局部变量
	SP_AntiDebug(iAntiDebugStatus);
		if (iCrcStatus)
	{
		printf("进程被调试!\n");
	}
	else
	{
		printf("进程没有被调试!\n");
	}
	
	SP_End();
	
	return 0;
}
```

10. 反调试水印`SP_ANTIDEBUG_AUTO`是一个用来检测动态调试的水印, 一旦发现被调试会立即自动结束进程(如果不需要上传被调试的日志, 推举你使用这个水印), 使用方法如下:
```C++
int main(void)
{
	SP_Begin(1);
	
	printf("hello, sp1!\n");
	printf("hello, sp2!\n");
	printf("hello, sp3!\n");
	
	SP_ANTIDEBUG_AUTO(1);	// 一旦发现被调试, 则自动结束进程
	
	SP_End();
	
	return 0;
}
```

11. 请多利用 代码加密水印`SP_Begin, SP_End` 跟 防止Dump代码加密水印`SP_AntiDumpCodeBegin, SP_AntiDumpCodeEnd`的组合

12. 请注意, MAP文件操作不支持 防止Dump代码加密

13. 请注意, 在同一个pe文件里面, 你不能`SP_ANTIDEBUG`跟`SP_ANTIDEBUG_AUTO`同时使用, 你只能选择其中一种方案, 否则会出现问题. (`SP_CRC`跟`SP_CRC_AUTO`也使用相同的规则)

14. 更多的使用DEMO请关注WEB站内公告
