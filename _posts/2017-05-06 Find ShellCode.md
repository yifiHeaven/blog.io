---
layout: post
title: Find Shellcode
date: 2017-05-06
categories: blog
tags: [�ܽ�,֪ʶ����]
description: 
---

/*
	
	(1) ���ڴ�������һ��jmp esp ָ��ĵ�ַ���Ǻ������ص�ַ��������ԭ�����ֹ������ShellCode��ʼ��ֱַ�Ӹ���
	(2) �������غ��ض���ȥִ���ڴ��е�����jmp espָ�������ֱ�ӿ�ʼִ��ShellCode
	(3) ����esp�ں�������ʱ��ָ��ջ�����������ص�ַ֮��,jmp esp ָ�ִ�к󣬴������ᵽջ���������ص�ַ
֮��ĵط�ȡִ��ָ��
	(4) ���²���ShellCode ����û�������ص�ַ֮�󣬼�����ûһƬջ�ռ䡣��������ǰ��һ�εط�������������䣬��
ShellCodeǡ�ðڷ��ں����������ص�ַ֮��������jmp espָ��ִ�й���ǡ������ShellCode
	Of Course ESP��ָ��λ�û��뺯���ĵ���Լ�������ص�ַ���й�
	���磺retn 3 �� retn 4�ڷ��غ�esp��ָ��λ�ö�����������
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
	����jmp esp ��Ӧ�Ļ����� 0xFFE4
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