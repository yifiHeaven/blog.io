---
layout: post
title: Find Shellcode
date: 2017-05-06
categories: blog
tags: [总结,知识管理]
description: 
---

/*
	
	(1) 在内存中任意一个jmp esp 指令的地址覆盖函数返回地址，而不是原来用手工查出的ShellCode起始地址直接覆盖
	(2) 函数返回后被重定向去执行内存中的这条jmp esp指令，而不是直接开始执行ShellCode
	(3) 由于esp在函数返回时仍指向栈区（函数返回地址之后）,jmp esp 指令被执行后，处理器会到栈区函数返回地址
之后的地方取执行指令
	(4) 重新布置ShellCode 在淹没函数返回地址之后，继续淹没一片栈空间。将缓冲区前边一段地方用任意数据填充，把
ShellCode恰好摆放在函数函数返回地址之后，这样，jmp esp指令执行过后恰好跳进ShellCode
	Of Course ESP所指的位置还与函数的调用约定、返回地址等有关
	例如：retn 3 与 retn 4在返回后，esp所指的位置都会有所差异
*/
    8: 	int a = 0;
0091171E C7 45 F8 00 00 00 00 mov         dword ptr [a],0  
     9: 	__asm {
    10: 		jmp esp
00911725 FF E4                jmp         esp  
    11: 		retn
00911727 C3                   ret  
    12: 	}

/*
	搜索jmp esp 对应的机器码 0xFFE4
*/
#include <iostream>
#include <Windows.h>
#define DLL_NAME L"user32.dll"
using namespace std;

int main()
{
	HANDLE DllHandle = LoadLibrary(DLL_NAME);
	BOOL Flag = FALSE;
	if (!DllHandle)
	{
		cout << "Load Dll Error" << endl;
		return 0;
	}
	CHAR* ptr = (CHAR*)DllHandle;
	for (int i = 0; !Flag; i++)
	{
		try {
			if (ptr[i] == 0xFF && ptr[i + 1] == 0xE4)
			{
				int Address = (int)ptr + i;
				printf("0x%x\r\n", Address);
			}
		}
		catch (...)
		{
			int Address = (int)ptr + i;
			printf("0x%x\r\n", Address);
			Flag = TRUE;
		}
	}

}