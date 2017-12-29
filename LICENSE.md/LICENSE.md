#include <windows.h>
#include "resource.h" //**********一定要加头文件， 要不然INB_BITMAP无法加载***********

HINSTANCE g_hInstance = 0; // 用来接收当前程序实例句柄
void DrawBMP(HDC hdc)
{
	//添加位图资源：不需要写代码，自己绘制一个位图 //******方法--:资源-->添加位图-->画区一张图片
	//2.加载位图
	//HBITMAP hBmap =  LoadBitmap(g_hInstance,(CHAR*)IDB_BITMAP1);
	HBITMAP hBmap =  LoadBitmap(g_hInstance, (CHAR*) IDB_BITMAP1);
	//3.创建一个和当前dc匹配的内存DC
	//同时在内存中构建一个虚拟区域，内存DC对应这个区域
	/*
	* 创建一个内存DC的内核结构，同时申请一块内存， 内存DC维护这块内存申请的空间和当前DC维护的显存的大小相同
	* 当前DC维护的显存的大小由窗口大小决定
	*/
	HDC hMendc =  CreateCompatibleDC(hdc);
	//4.将创创建好的位图数据放入内存DC中，内存DC立即在虚拟内存中绘制位图
	/*
	* 把位图数据保存在内存DC所维护的那块内存中
	*/
	HGDIOBJ nOldBmp = (hMendc, hBmap);

	//BITMAP bmpInfo = {0};
	//GetObject(hBmap, sizeof(bmpInfo), &bmpInfo);
	//5.绘制位图（成像（1：1））
	/*
	* 相当于将内存DC维护的那块内存中的数据，以拷贝内存的方式一次性的复制到当前DC所维护的显存中
	*/
	//BitBlt(hdc, 30, 30, bmpInfo.bmWidth, bmpInfo.bmHeight,hMendc, 0, 0, SRCCOPY);
	StretchBlt(hdc, 60, 60, 260, 260, hMendc, 0, 0, 100, 100, SRCCOPY);
	//6.从内存DC中取回位图
	SelectObject(hMendc, nOldBmp);
	//7.释放位图
	DeleteObject(hBmap);
	//8.释放内存DC
	DeleteDC(hMendc);
}
void  onPaint(HWND hWnd)
{
	PAINTSTRUCT ps = {0};
	HDC hdc = BeginPaint(hWnd, &ps);
	//绘图
	DrawBMP(hdc);
	EndPaint(hWnd,&ps);
}
//2.窗口处理函数
LRESULT CALLBACK WndProc(HWND hWnd, UINT msgID, WPARAM wParam, LPARAM lparam)
{
	switch(msgID)
	{
	case WM_PAINT:
		onPaint(hWnd);
		break;
	case WM_DESTROY:
		PostQuitMessage(0);
		break;
	}
	return DefWindowProc(hWnd, msgID, wParam, lparam);//给各种消息进行默认处理
}
//注册窗口类
void Register(LPSTR lpClassName, WNDPROC)
{
	WNDCLASSEX wce = {0};
	wce.cbSize = sizeof(wce);//结构体大小
	wce.cbClsExtra = 0;//窗口类附加数缓冲区
	wce.cbWndExtra = 0;//窗口附加数据缓冲区
	wce.hbrBackground = (HBRUSH) (COLOR_WINDOW + 1);//背景颜色
	wce.hIcon = NULL;//图标
	wce.hCursor = NULL;//鼠标
	wce.hIconSm = NULL;//大图标
	wce.hInstance = g_hInstance;//句柄
	wce.lpfnWndProc = WndProc;//调用程序
	wce.lpszClassName = lpClassName;//窗口名字
	wce.lpszMenuName = NULL;//菜单
	wce.style = CS_HREDRAW|CS_HREDRAW;//窗口风格
	RegisterClassEx(&wce);
}
//4.创建窗口
HWND CreateMain(LPSTR lpClassName, LPSTR lpWndName)
{
	HWND hWnd = CreateWindowEx(0, lpClassName, lpWndName, WS_OVERLAPPEDWINDOW, 100, 100, 700, 500, NULL, NULL, g_hInstance, NULL);
	return hWnd;
}
//5.显示窗口
void Display(HWND hWnd)
{
	ShowWindow(hWnd, SW_SHOW);
	UpdateWindow(hWnd);
	////刷新窗口，不调用也没有关系，但是官网推荐调用

}
//6.显示循环
void Message()
{
	MSG nMsg = {0};
	while(GetMessage(&nMsg, NULL,0, 0))
	{
		TranslateMessage(&nMsg);
		DispatchMessage(&nMsg);
	}
}
//1.入口函数
int CALLBACK WinMain(HINSTANCE hIns, HINSTANCE hPreIns, LPSTR lpCmdLine, int nCmdShow)
{
	g_hInstance  = hIns;
	Register("Main", WndProc);
	HWND hWnd = CreateMain("Main", "Window");
	Display(hWnd);
	Message();
	return 0;
}
