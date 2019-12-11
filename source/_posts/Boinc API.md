---
title: Boinc API
date: 2019-12-09 18:28:48
tags: Boinc
---

# Boinc C++ 任务 API 

<!-- more -->

​		主要是对[Boinc 文档](https://boinc.berkeley.edu/trac/wiki/BasicApi)的一些笔记。

​		Boinc 提供了 `boinc_api.h` 大多都有C接口，因此可以移植给别的语言（通过其他语言的c/c++扩展机制，如nodejs python), API协定返回整数错误码，除0之外的所有值应该被是为失败。

# 核心API

### 初始化

```c++
boinc_init()
```



​		函数主要作用是在内部判断是否已经初始化，如果是多线程就准备好线程，初始化内部变量，重定向输出，等等准备工作

### 多线程或多进程

```c++
// 初始化 option 结构
static BOINC_OPTIONS options;
boinc_options_defaults(options) // 函数会给option赋值
// 如果是只是用多线程,默认是使用多进程
options.multi_thread = true;
// 准备需要用到的线程
boinc_init_options(&option)
```

​		在任何时候创建多进程或者多线程的时候可以调用上面的函数进行创建。

### 非CPU计算

```c++
// 初始化 option 结构
static BOINC_OPTIONS options;
boinc_options_defaults(options);
options.normal_thread_priority = true;
boinc_init_options(&options);
```

​		赋值之后会调用windows提供的API降低当前创建线程的优先级



### 结束

```c++
int boinc_finish(int state)
```

​		boinc_finish 函数会根据 state的值来确定是否结束当前任务，如果 state是0那么boinc会结束任务，并返回0，如果state是非零那么boinc会重启任务的应用程序

```c++
boinc_finish_message(int status, const char* msg, bool is_notice);
```

​		调用此API会向GUI中发送消息。

### 解析文件路径	

```c++
int boinc_resolve_filename(char *logical_name, char *physical_name, int len);
```

​		有时需要读取本地文件（例如任务配套的数据文件），但是因为无法预知路径，所以Boinc提供了这个api，接受名称和和一个char *类型字符串指针和长度，之后函数会将地址赋值给 physical_name 以供函数使用



### IO包装函数

```c++
boinc_fopen(char* path, char* mode);
```

​		Boinc提供了一些文件操作函数，函数处理了一些问题，例如windows一些文件操作会锁定文件，Boinc提供了API来处理这些问题。用来替换例如`fopen`之类的函数



### 检查点

```c++
int boinc_time_to_checkpoint()
int boinc_checkpoint_completed()
```

​		程序可能需要在本地保存一些当前计算状态，在暂停或者重启或者其他异常状态中恢复到一个就近的状态，Boinc对此提供了`boinc_time_to_checkpoint`函数，用于检查当前是否合适进行文件独写，如果返回`非零` 则程序应该立即写入文件，然后调用`boinc_checkpoint_completed `函数来告知

Boinc写入完毕, `boinc_time_to_checkpoint`自身运行速度很快，可以频繁进行检查（每秒成百上千次调用），但是这个API仅在上一次检查并写入之后（调用`boinc_checkpoint_completed `会重置时间戳）`足够长的时间`

​		这个足够长的时间取决于:

1. 用户的首选项(例如笔记本用户可能不希望检查点)
2. 由调用指定的可选应用程序提供的

```c++
boinc_set_min_checkpoint_period(int nsecs)
```

​		函数可以用来设置检查点写入间隔

​		一般写入的方法是:

1. 调用`boinc_time_to_checkpoint`函数检查是否可以写入
2. 一般使用下面的文件类的flush 或者 close 函数来写入
3. 然后调用`boinc_checkpoint_completed`
4. 如果不可以写入那么跳过本次写入数据会保留在内存里



### 保护核心代码

```c++
void  boinc_begin_critical_section(); 
void  boinc_end_critical_section();
```

​		如果想保护某些代码的顺利执行而不会被挂起或者中断,在开始执行关键代码的时候调用`boinc_begin_critical_section` 执行完毕的时候调用 `boinc_end_critical_section`,可以重复调用函数,但是函数必须两两匹配.



### 文件类

```c++
class MFILE {
public:
    int open(char* path, char* mode);
    int _putchar(char);
    int puts(char*);
    int printf(char* format, ...);
    size_t write(const void* buf, size_t size, size_t nitems);
    int close();
    int flush();
};
```

​		Boinc提供了一个 class 用于打开一个文件,将要写入的文件缓存在内存中,之后调用 `flush`或者`close`函数,之后才会将数据写入具体文件内,例如在上面的检查点函数中,如果检查通过则调用flush函数写入文件如果检查不通过,那么数据还是会在内存中储存.



### 进度

```c++
boinc_fraction_done(double frachtion_done)
```

​		该函数可告知Boinc管理器当前任务的任务进度,可以定期调用以更新进度,函数参数是一个 double 类型的小数,`0>=frachtion_done<=1`, 数值应该是递增的而不应该减少,如果程序出错需要重置应该调用`boinc_finish()`函数并传入一个非零值来使任务重启.



### 时间信息

```c++
int boinc_wu_cpu_time(double &cpu_time)
```

​		获取CPU时间总数(从开始执行任务开始,重启任务时间不会重置

```c++
int boinc_elapsed_time()
```

​		该函数会返回当前任务执行时间(任务运行所使用的时间,不包括暂停重启等其他)

### 运行模式

```c++
int boinc_is_standalone();
```

​		应用程序可以在独立模式下运行进行,也可以在Boinc客户端控制下运行,如果要确认当前运行模式可以使用该函数,如果程序是独立模式则返回`非零`否则返回`零`



### 定时器

```c++
typedef void (*FUNC_PTR)();
void boinc_register_timer_callback(FUNC_PTR);
```

​		`boinc_register_timer_callback`函数是一个定时器, 通过给其传递一个函数指针,FUNC_PTR函数会被每秒调用一次



### 退出并重启任务

```c++
int boinc_temporary_exit(int delay, const char* reason=NULL, bool is_notice=false);
```

​		如果程序遇到错误需要退出并重启任务,则需要调用这个函数,告诉Boinc在几秒之后重启应用,如果`is_notice = true` 则会通知用户发生错误并显示`reason`

​		例如程序需要GPU RAM,来运行程序,但是Boinc无法为程序分配GPU RAM,则尝试过一会重启任务.



## 跨平台实用功能



### 杂项

​		函数都分别声明和定义在于 lib/util.cpp/h 中

```c++
double dtime(); // 返回一个 Unix时间(以小数表示)
double dday(); // 返回这一天开始的Unix时间
void boinc_sleep(double); // sleep 指定秒数
double drand(); // 返回一个伪随机的函数 
double rand_normal(); // 返回一个真随机的函数
read_file_string(const char* path, string&); // 将指定文件路径的文件读进字符串里面
```



### 文件系统函数

​		关于文件系统的一些操作都定义在 lib/filesys.cpp/h 中

```c++
 extern int boinc_delete_file(const char*);
  extern int boinc_touch_file(const char *path);
  extern FILE* boinc_fopen(const char* path, const char* mode);
  extern int boinc_copy(const char* orig, const char* newf);
  extern int boinc_rename(const char* old, const char* newf);
  extern int boinc_mkdir(const char*);
#ifdef _WIN32
  extern int boinc_allocate_file(const char*, double size);
#else
  extern int boinc_chown(const char*, gid_t);
#endif
  extern int boinc_rmdir(const char*);
  extern void boinc_getcwd(char*);
  extern void relative_to_absolute(const char* relname, char* path);
  extern int is_file(const char* path);
  extern int is_dir(const char* path);
  extern int is_file_follow_symlinks(const char* path);
  extern int is_dir_follow_symlinks(const char* path);
  extern int is_symlink(const char* path);
  extern bool is_dir_empty(const char*);
  extern int boinc_truncate(const char*, double);
  extern int boinc_file_exists(const char* path);
  extern int boinc_file_or_symlink_exists(const char* path);
  extern int file_size(const char*, double&);
  extern int clean_out_dir(const char*);
  extern int dir_size(const char* dirpath, double&, bool recurse=true);

  class DirScanner;   // 扫描目录
  struct FILE_LOCK;   // 文件锁
```



### XML解析

​		Boinc 使用XML来记录数据,所以提供一个类来处理方法都在lib/parse.cpp/h内

```c++
bool boinc_is_finite(double);
int copy_element_contents(FILE* in, const char* end_tag, char* p, size_t len);
int copy_element_contents(FILE* in, const char* end_tag, std::string&);
int copy_stream(FILE* in, FILE* out);
int dup_element(FILE* in, const char* end_tag, char** pp);
int dup_element_contents(FILE* in, const char* end_tag, char** pp);
void extract_venue(const char*, const char*, char*, int len);
bool match_tag(const char* buf, const char* tag);
bool match_tag(const std::string &s, const char* tag);
void non_ascii_escape(const char*, char*, int len);
bool parse(char* , char* );
void parse_attr(const char* buf, const char* attrname, char* out, int len);
bool parse_bool(const char*, const char*, bool&);
bool parse_double(const char* buf, const char* tag, double& x);
bool parse_int(const char* buf, const char* tag, int& x);
bool parse_str(const char* buf, const char* tag, std::string& dest);
bool parse_str(const char*, const char*, char*, int);
bool remove_element(char* buf, const char* start, const char* end);
void replace_element_contents(
    char* buf, const char* start, const char* end, const char* replacement
);
char* sgets(char* buf, int len, char* &in);

int skip_unrecognized(char* buf, MIOFILE&);
bool str_replace(char* str, const char* old, const char* neww);
int strcatdup(char*& p, char* buf);
void xml_unescape(char*);
void xml_unescape(std::string&);
```

​		XML_PARSER 是一个类(结构体),提供了各种各样对XML使用的实用方法.比较抽象,举个例子说明一下.



```C++
FILE* f = boinc_fopen("test.xml", "r"); // 打开文件
MIOFILE mf; // 声明一个文件的实例
mf.init_file(f);  // 初始化实例
XML_PARSER xp(&mf); // 获得实例
while (!xp.get_tag()) {
        if (xp.parse_string("user_name", user_name)) {
            continue;
        } else if (xp.parse_string("team_name", team_name)) {
            continue;
        } else if (xp.parse_string("authenticator", authenticator)) {
            continue;
        } else if (xp.parse_string("error_msg", error_msg)) {
            continue;
        }
    }
```



### 字符串处理函数

​		函数定义在 lib.str_util.cpp/h

​		跨平台支持定义在  strlcpy(), strlcat(), strcasestr() and strcasecmp() is in lib/str_replace.cpp,h 

```c++
const char* active_task_state_string(int state);
const char* batch_state_string(int state);
const char* battery_state_string(int state);
const char* boincerror(int which_error); //  Return a text-string description of a given error.Must be kept consistent with error_numbers.h
void collapse_whitespace(char *str);
void collapse_whitespace(string& str);
bool is_valid_filename(const char* name);
char* lf_terminate(char* p);
void mysql_timestamp(double dt, char* p);
void nbytes_to_string(double nbytes, double total_bytes, char* str, int len); // Convert nbytes into a string.  If total_bytes is non-zero,convert the two into a fractional display (i.e. 4/16 KB)


// Converts a double precision time (where the value of 1 represents
// a day) into a string.  smallest_timescale determines the smallest
// unit of time division used
// smallest_timescale: 0=seconds, 1=minutes, 2=hours, 3=days, 4=years
//
int ndays_to_string (double x, int smallest_timescale, char *buf);

const char* network_status_string(int n);

int parse_command_line(char* p, char** argv);

void parse_serialnum(char* in, char* boinc, char* vbox, char* coprocs);

// get the name part of a filepath
//
// wrapper for path_to_filename(string, string&)
int path_to_filename(string fpath, char* &fname);


// get the name part of a filepath
// returns:
//   0 on success
//  -1 when fpath is empty
//  -2 when fpath is a directory
int path_to_filename(string fpath, string& fname);


char* precision_time_to_string(double t);

void remove_str(char* p, const char* str);

const char* result_client_state_string(int state);

const char* result_scheduler_state_string(int state);

const char* rpc_reason_string(int reason);

const char* run_mode_string(int mode);

void secs_to_hmsf(double secs, char* buf);

vector<string> split(string s, char delim);

// BOINC only uses strcasestr() for short strings,
// so the following will suffice
//
const char *strcasestr(const char *s1, const char *s2);

// version of strcpy that works even if strings overlap (p < q)
//
void strcpy_overlap(char* p, const char* q);

// string substitution:
// haystack is the input string
// out is the output buffer
// out_len is the length of the output buffer
// needle is string to search for within the haystack
// target is string to replace with
//
int string_substitute(
    const char* haystack, char* out, int out_len,
    const char* needle, const char* target
);

void strip_quotes(char *str);


// remove whitespace and quotes from start and end of a string
//
void strip_quotes(string& str);

// remove _(" and ") from string
//
void strip_translation(char* p);

void strip_whitespace(char *str);

// remove whitespace from start and end of a string
//
void strip_whitespace(string& str);


size_t strlcat(char *dst, const char *src, size_t size);

// Use this instead of strncpy().
// Result will always be null-terminated, and it's faster.
// see http://www.gratisoft.us/todd/papers/strlcpy.html
//
#if !HAVE_STRLCPY
size_t strlcpy(char *dst, const char *src, size_t size);

const char* suspend_reason_string(int reason);


char* time_to_string(double t);

string timediff_format(double diff);

// This only unescapes some special shell characters used in /etc/os-release
// see https://www.freedesktop.org/software/systemd/man/os-release.html
void unescape_os_release(char* buf);
```

​		其他就不一一列举了.





### RSA 加密

​		详见 lib/crypt.cpp/h



### MD5 hash

​		详见 lib/md5.cpp/h



### sempahores

​		详见 lib.synch.app/h



### 网络通信

​		详见 lib/network.cpp/h

```c++
void boinc_close_socket(int sock);
    
// Get an unused port number.
// Used by vboxwrapper.
// I'm not sure if is_remote is relevant here - a port is a port, right?
//
int boinc_get_port(bool is_remote, int& port);
int boinc_socket(int& fd, int protocol);

int boinc_socket_asynch(int fd, bool asynch);


int get_socket_error(int fd);

bool is_localhost(sockaddr_storage& s);

void reset_dns();

int resolve_hostname(const char* hostname, sockaddr_storage &ip_addr);

int resolve_hostname_or_ip_addr(
    const char* hostname, sockaddr_storage &ip_addr
);

bool same_ip_addr(sockaddr_storage& s1, sockaddr_storage& s2);


const char* socket_error_str();

int WinsockCleanup();

int WinsockInitialize();
```



### 进程管理

​		详见lib/proc_control.cpp/h

```c++
// return a list of all descendants of the given process
//
void get_descendants(int pid, vector<int>& pids);
static void get_descendants_aux(PROC_MAP& pm, int pid, vector<int>& pids) ;
void kill_all(vector<int>& pids);

// Kill the descendants of the calling process.
void kill_descendants();

// return OS-specific value associated with priority code
//
int process_priority_value(int priority);


// suspend/resume the descendants of the calling process
// (or if pid==0, the calling process)
//
void suspend_or_resume_descendants(bool resume);

// used by the wrapper
//
void suspend_or_resume_process(int pid, bool resume);


// Suspend or resume the threads in a set of processes,
// but don't suspend 'calling_thread'.
//
// Called from:
//  API (processes = self)
//      This handles a) throttling; b) suspend/resume by user
//  wrapper (via suspend_or_resume_process()); process = child
//  wrapper, MP case (via suspend_or_resume_decendants());
//      processes = descendants
//
// The only way to do this on Windows is to enumerate
// all the threads in the entire system,
// and identify those belonging to one of the processes (ugh!!)
//
int suspend_or_resume_threads(
    vector<int>pids, DWORD calling_thread_id, bool resume, bool check_exempt
) 
```

