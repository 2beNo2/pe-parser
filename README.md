# Pe File Parser
- 熟悉Pe文件结构，程序只支持32位
- PE格式中重要的数据目录：
  - **导出表**：了解导出表，可以自己实现GetProcAddress()、GetProcFunName()等方法
  ```
  /*
	typedef struct _IMAGE_EXPORT_DIRECTORY {
	  DWORD   Characteristics;            
	  DWORD   TimeDateStamp;          
	  WORD    MajorVersion;               
	  WORD    MinorVersion;           
	  DWORD   Name;                   // dll名称
	  DWORD   Base;                   // 序号查询时会用上，数组的坐标平移
	  DWORD   NumberOfFunctions;      // 有多少个被导出的项    
	  DWORD   NumberOfNames;          // 有多少个被名称导出的项
	  DWORD   AddressOfFunctions;     // 导出地址表，rva
	  DWORD   AddressOfNames;         // 导出名称表，rva
	  DWORD   AddressOfNameOrdinals;  // 导出序号表，rva
	} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
	*/
  ```
  
  - **导入表**：了解导入表，加壳/脱壳都需要对导入表进行操作，可利用导入表进行注入
  ```
  typedef struct _IMAGE_IMPORT_DESCRIPTOR {
	    union {
	        DWORD   Characteristics;            
	        DWORD   OriginalFirstThunk;         // 导入名称表(INT)的RVA，GetProAddress
	    } DUMMYUNIONNAME;
	    DWORD   TimeDateStamp;                  // 忽略
	    DWORD   ForwarderChain;                 // 忽略
	    DWORD   Name;                           // dll名称的地址，LoadLibrary
	    DWORD   FirstThunk;                     // 指向导入地址表(IAT)的RVA，pfn填在此处
	} IMAGE_IMPORT_DESCRIPTOR;
	typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
  ```
  	`INT表`和`IAT表`都是`IMAGE_THUNK_DATA`结构的
  ```
  typedef struct _IMAGE_THUNK_DATA32 {
	  union {
	    PBYTE  ForwarderString;                 //转发字符串的RVA；
	    PDWORD Function;                        //导入函数的地址；
	    DWORD Ordinal;                          //导入函数的序号；
	    PIMAGE_IMPORT_BY_NAME  AddressOfData;   //指向IMAGE_IMPORT_BY_NAME；
	  } u1;
	} IMAGE_THUNK_DATA32;

	// IMAGE_THUNK_DATA32 在不同的状态下有不同的解释方式：
	// 在文件状态下解释为 PIMAGE_IMPORT_BY_NAME
	// 进程状态后是函数地址
	// 如果是序号导入的函数，最高位应该为一，取LWORD作为序号

	typedef struct _IMAGE_IMPORT_BY_NAME {
	  WORD Hint;     // 编译器添加的当前电脑中对应函数的序号
	  BYTE Name[1];  // 字符串
	} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
  ```
  
  - **重定位表**：LoadPe和加壳器，需要对重定位表中的数据做重定位处理
  ```
  typedef struct _IMAGE_BASE_RELOCATION {
	    DWORD   VirtualAddress;  // 页起始地址RVA，通知系统该分页上有数据需要重定位
	    DWORD   SizeOfBlock;     // 整个数据块的大小，包含SizeOfBlock
	//  WORD    TypeOffset[1];   // 柔性数组，保存了要修正的数据相对于页的偏移
				     // 低12位表偏移
			             // 数组成员的高4位，决定了修复的方式，是修正4个字节还是2个字节
				     // 高4位为0，表示无效，用来对齐
			             // 高4位为3，表示修4字节
			             // 高4位为0xA，表示修8字节
	} IMAGE_BASE_RELOCATION;
	typedef IMAGE_BASE_RELOCATION UNALIGNED * PIMAGE_BASE_RELOCATION;
  ```
  
  
  - **资源表**：
  
  
