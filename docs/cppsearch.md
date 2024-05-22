# C++搜索引擎项目

## String的实现

```c++
/* String.cc */
#include <string.h>
#include <iostream>

using std::cout;
using std::endl;

//面试的题目
class String
{
public:
    String()
    : _pstr(nullptr)
    {
        cout << "String()" << endl;
    }
    String(const char *pstr)
    : _pstr(new char[strlen(pstr) + 1]())
    {
        cout << "String(const char *)" << endl;
        strcpy(_pstr, pstr);
    }

    String(const String & rhs)
    : _pstr(new char[strlen(rhs._pstr) + 1]())
    {
        cout << "String(const String &)" << endl;
        strcpy(_pstr, rhs._pstr);
    }

    String & operator=(const String & rhs)
    {
        cout << "String &operator=(const String &)" << endl;
        //1、自复制
        if(this != &rhs)
        {
            //2、释放左操作数
            if(_pstr)
            {
                delete [] _pstr;
                _pstr = nullptr;
            }

            //3、深拷贝
            /* if(rhs._pstr) */
            _pstr = new char[strlen(rhs._pstr) + 1]();
            strcpy(_pstr, rhs._pstr);
        }
        //4、返回*this
        return *this;
    }

    size_t length() const
    {
        size_t len = 0;
        if(_pstr)
        {
            len = strlen(_pstr);
        }

        return len;

    }

    const char * c_str() const
    {
        if(_pstr)
        {
            return _pstr;
        }
        else
        {
            return nullptr;
        }
    }


    ~String()
    {
        cout << "~String()" << endl;
        if(_pstr)
        {
            delete [] _pstr;
            _pstr = nullptr;
        }
    }

    void print() const
    {
        if(_pstr)
        {
            cout << "_pstr = " << _pstr << endl;
        }
    }

private:
    char * _pstr;
};

int main(void)
{
    String str1;
    str1.print();

    cout << endl << endl;
    //C++         C,"Hello,world"===>String("Hello,world")
    String str2 = "Hello,world";
    String str3("wangdao");

    str2.print();		
    str3.print();	

    cout << endl << endl;
    String str4 = str3;
    str4.print();

    cout << endl << endl;
    str4 = str2;
    str4.print();

    //从C++风格字符串转换为C风格字符串
    const char *pstr = str3.c_str();

    return 0;
}
```

## 词频统计的Vector实现

```c++
/* dictVector.cc */
#include <iostream>
#include <string>
#include <vector>
#include <fstream>
#include <sstream>

using std::cout;
using std::endl;
using std::cerr;
using std::string;
using std::vector;
using std::ifstream;
using std::ofstream;
using std::istringstream;

struct Record
{
    Record(const string &word, int frequency)
    : _word(word)
    , _frequency(frequency)
    {

    }
	string _word;
	int _frequency;
};

class Dictionary
{
public:
    Dictionary(int capa)
    {
        _dict.reserve(capa);//预留空间
    }

    void read(const string &filename)
    {
        ifstream ifs(filename);
        if(!ifs.good())
        {
            cerr << "open " << filename << " error" << endl;
            return;
        }
        //读filename这个文件，然后对每一个单词做处理
        string line;
        while(getline(ifs, line))
        {
            //字符串IO
            istringstream iss(line);
            string word;
            while(iss >> word)
            {
                //word一定是最终要保存的单词吗？hello!  1
                string newWord = dealWord(word);//对老的单词做处理，得到新的单词
                insert(newWord);//将满足条件的单词存在vector中
            }
        }

        ifs.close();
    }

    void store(const string &filename)
    {
        ofstream ofs(filename);
        if(!ofs.good())
        {
            cerr << "open " << filename << " error!" << endl;
            return;
        }

        for(size_t idx = 0; idx != _dict.size(); ++idx)
        {
            ofs << _dict[idx]._word << "      " 
                << _dict[idx]._frequency << endl;
        }

        ofs.close();
    }

    string dealWord(const string &word)
    {
        for(size_t idx = 0; idx != word.size(); ++idx)
        {
            //word!
            /* if((word[idx] >= 'a'&& word[idx] <= 'z') || (word[idx] >= 'A'&& word[idx] <= 'Z')) */
            if(!isalpha(word[idx]))
            {
                return string();//临时对象
            }
        }

        return word;
    }

    void insert(const string &word)
    {
        //判断空串
        if(word == string())
        {
            return ;
        }

        size_t idx = 0;
        for(idx = 0; idx != _dict.size(); ++idx)
        {
            if(word == _dict[idx]._word)
            {
                ++_dict[idx]._frequency;//频率进行++
                break;
            }
        }

        if(idx == _dict.size())
        {
            _dict.push_back(Record(word, 1));//单词第一次出现的代码
        }
    }

private:
	vector<Record> _dict;
};

int main(int argc, char **argv)
{
    Dictionary dictionary(13000);
    cout << "begin reading..." << endl;
    dictionary.read("The_Holy_Bible.txt");
    cout << "end reading..." << endl;
    dictionary.store("dict.dat");
    return 0;
}
```

## String的运算符重载

```c++
/* stringOperator.cc */
#include <string.h>
#include <iostream>
#include <vector>

using std::cout;
using std::endl;
using std::vector;

class String 
{
public:
    String()
   // : _pstr(nullptr)//后面操作的时候，需要判空
	: _pstr(new char[1]())
    {
        cout << "String()" << endl;
    }

    //String s1("hello")
    //String s1 = "hello";//String("hello")
    //"hello"=====String("hello")
    String(const char *pstr)
    : _pstr(new char[strlen(pstr) + 1]())
    {
        cout << "String(const char *)" << endl;
        strcpy(_pstr, pstr);
    }

    //String s2(s1);
    //String s2 = s1;
    String(const String &rhs)
    : _pstr(new char[strlen(rhs._pstr) +1]())
    {
        cout << "String(const String &)" << endl;
        strcpy(_pstr, rhs._pstr);
    }

    ~String()
    {
        cout << "~String()" << endl;
        if(_pstr)
        {
            delete [] _pstr;
            _pstr = nullptr;
        }
    }

    //String s1;
    //s1 = s1;
    String &operator=(const String &rhs)
    {
        cout << "String &operator=(const String &)" << endl;
        if(this != &rhs)
        {
            delete [] _pstr;
            _pstr = nullptr;

            _pstr = new char[strlen(rhs._pstr) + 1]();
            strcpy(_pstr, rhs._pstr);
        }
        return  *this;
    }

    // s1 = "hello";增量开发
    String &operator=(const char *pstr)
    {
        cout << "String &operator=(const char *)" << endl;
        String tmp(pstr);
        *this = tmp;

        return *this;
    }

    //s1 += s2;
    String &operator+=(const String &rhs)
    {
        cout << "String &operator+=(const String &)" <<endl;
        String tmp;
        if(tmp._pstr)
        {
            delete [] tmp._pstr;//防止内存泄漏
        }
        tmp._pstr = new char[strlen(_pstr) + 1]();
        strcpy(tmp._pstr, _pstr);
        delete [] _pstr;
        _pstr = nullptr;
        _pstr = new char[strlen(rhs._pstr) + strlen(tmp._pstr) + 1]();
        strcpy(_pstr, tmp._pstr);
        strcat(_pstr, rhs._pstr);

        return *this;
    }

    //s1 += "hello"
    String &operator+=(const char *pstr)
    {
        cout << "String &operator+=(const char *)" << endl;
        String tmp(pstr);
        *this += tmp;

        return *this;
    }

    //const String s1("helo");
    //s1[0]
    char &operator[](std::size_t index)//index > = 0
    {
        if(index < size())
        {
            return _pstr[index];
        }
        else
        {
            static char nullchar = '\0';
            return nullchar;
        }
    }

    const char &operator[](std::size_t index) const
    {
        if(index < size())
        {
            return _pstr[index];
        }
        else
        {
            static char nullchar = '\0';
            return nullchar;
        }

    }

    std::size_t size() const
    {
        return strlen(_pstr);
    }

    const char* c_str() const
    {
        return _pstr;
    }

    friend bool operator==(const String &, const String &);
    friend bool operator!=(const String &, const String &);

    friend bool operator<(const String &, const String &);
    friend bool operator>(const String &, const String &);
    friend bool operator<=(const String &, const String &);
    friend bool operator>=(const String &, const String &);

    friend std::ostream &operator<<(std::ostream &os, const String &s);
    friend std::istream &operator>>(std::istream &is, String &s);

private:
    char * _pstr;
};

bool operator==(const String &lhs, const String &rhs)
{
    return !strcmp(lhs._pstr, rhs._pstr);
}

bool operator!=(const String &lhs, const String &rhs)
{
    return strcmp(lhs._pstr, rhs._pstr);
}

bool operator<(const String &lhs, const String &rhs)
{
    return strcmp(lhs._pstr, rhs._pstr) < 0;
}

bool operator>(const String &lhs, const String &rhs)
{
    return strcmp(lhs._pstr, rhs._pstr) > 0;
}
bool operator<=(const String &lhs, const String &rhs)
{
    return strcmp(lhs._pstr, rhs._pstr) <= 0;
}
bool operator>=(const String &lhs, const String &rhs)
{
    return strcmp(lhs._pstr, rhs._pstr) >= 0;
}

std::ostream &operator<<(std::ostream &os, const String &rhs)
{
    if(rhs._pstr)
    {
        os << rhs._pstr;
    }

    return os;
}

//String s1("hello")
//cin >> s1;
std::istream &operator>>(std::istream &is, String &rhs)
{
    if(rhs._pstr)
    {
        delete [] rhs._pstr;
        rhs._pstr = nullptr;
    }

    //动态获取从键盘输入数据的长度
    vector<char> buffer;
    char ch;
    while((ch = is.get()) != '\n')
    {
        buffer.push_back(ch);
    }

    rhs._pstr = new char[buffer.size() + 1]();
    strncpy(rhs._pstr, &buffer[0], buffer.size());

    return is;
}

String operator+(const String &lhs, const String &rhs)
{
    cout << "String operator+(const String &, const String &)" << endl;

    String tmp(lhs);
    tmp += rhs;

    return tmp;
}
//s1 + "hello"
String operator+(const String &lhs, const char *pstr)
{
    cout << "String operator+(const String &, const char *)"<< endl;
    String tmp(lhs);
    tmp += pstr;

    return tmp;

}

//"hello" + s1
String operator+(const char *pstr, const String &rhs)
{
    cout << "String operator+(const char*, const String &)" << endl;
    String tmp(pstr);
    tmp += rhs;

    return tmp;
}


void test()
{
    String s1;
    /* std::cin >> s1; */
    cout << "s1 = " << s1 << endl;

    cout << endl << endl;
    String s2 = "hello";
    cout << "s2 = " << s2 << endl;

    cout << endl << "1111" <<  endl;
    s2 = "world"; //error
    cout << "s2 = " << s2 << endl;

    cout << endl << endl;
    s2 = s2;
    cout << "s2 = " << s2 << endl;

    cout << endl << endl;
    String s3 = "wuhan";
    s3 += " welcome to string word";
    cout << "s3 = " << s3 << endl;
}

int main(int argc, char **argv)
{
    test();
    return 0;
}
```

## 词频统计Map实现及其与Vector对比

### Map实现

```c++
#include <time.h>
#include <iostream>
#include <string>
#include <fstream>
#include <sstream>
#include <map>
#include <utility>

using std::endl;
using std::cerr;
using std::cout;
using std::string;
using std::ifstream;
using std::ofstream;
using std::istringstream;
using std::map;
using std::pair;

class Dictionary
{
public:
    void read(const string &filename)
    {
        ifstream ifs(filename);
        if(!ifs)
        {
            cerr << "ifs open " << filename << " error!" << endl;
            return;
        }

        string line;
        while(getline(ifs, line))
        {
            istringstream iss(line);
            string word;
            while(iss >> word)
            {
                string newWord = dealWord(word);
                if(string() != newWord)
                {
                    ++_map[newWord];
                }
            }
        }

        ifs.close();
    }

    void store(const string &filename)
    {
        ofstream ofs(filename);
        if(!ofs)
        {
            cerr << "ofs open " << filename << " error!" << endl;
            return;
        }

        map<string, int>::iterator it;
        for(it = _map.begin(); it != _map.end(); ++it)
        {
            ofs << it->first << "  " << it->second << endl;
        }

        ofs.close();
    }
private:
    string dealWord(const string &word)
    {
        //查看获取到的字符串是不是单词：标点符号，true1都不算
        for(size_t idx = 0; idx != word.size(); ++idx)
        {
            if(!isalpha(word[idx]))
            {
                //如果存在标点，数字等不算单词，返回空串
                return string();
            }
        }

        //转换为合理的单词
        return word;
    }

private:
    map<string, int> _map;
};

int main(void)
{
    cout << "before reading..." << endl;
    Dictionary dictionary;
    time_t beg = time(NULL);
    dictionary.read("The_Holy_Bible.txt");
    time_t end = time(NULL);
    cout << "time: " << (end - beg) << "s" << endl;
    cout << "aftre reading..." << endl;
    dictionary.store("dictMap.dat");
    return 0;
}
```

### Vector实现

```c++
#include <time.h>
#include <iostream>
#include <string>
#include <vector>
#include <fstream>
#include <sstream>
#include <algorithm>

using std::cout;
using std::cerr;
using std::endl;
using std::string;
using std::vector;
using std::ifstream;
using std::ofstream;
using std::istringstream;
using std::sort;

struct Record
{
    Record(const string &word, int frequency)
    : _word(word)
    , _frequency(frequency)
    {
    }

    string _word;
    int _frequency;
};

bool operator<(const Record &lhs, const Record &rhs)
{
    return lhs._word < rhs._word;
}

class Dictionary
{
public:
    Dictionary(int capa)
    {
        _dict.reserve(capa);
    }

    void read(const string &filename)
    {
        ifstream ifs(filename);
        if(!ifs)//bool operator!(){}  operator bool() {}
        {
            //cout cerr clog
            cerr << "ifs open file " << filename << " error!" << endl;
            return;
        }

        string line;
        //while(ifs >> word)
        while(getline(ifs, line))
        {
            istringstream iss(line);//串IO，内存
            string word;

            //vector<Point> pt = {Point(1, 2), {3, 4}, (5, 6)};
            //逗号表达式
            while(iss >> word, !iss.eof())//word可能就是不规范abc123
            /* while(iss >> word)//word可能就是不规范abc123,while(真值表达式) iss  ---> bool /int */
            {
                //newWord是处理之后的新单词
                string newWord = dealWord(word);

                //把新的单词插入到vector里面
                insert(newWord);
            }
        }

        //将元素进行排序
        sort(_dict.begin(), _dict.end());

        ifs.close();
    }

    //将单词与词频存储到文件中
    void store(const string &filename)
    {
        ofstream ofs(filename);
        if(!ofs)
        {
            cerr << "ofs open " << filename << " error" << endl;
            return;
        }

        for(size_t idx = 0; idx != _dict.size(); ++idx)
        {
            ofs << _dict[idx]._word << "   " 
                << _dict[idx]._frequency << endl;
        }

        ofs.close();
    }

    //对不符合要求的单词进行处理
    string dealWord(const string &word)
    {
        for(size_t idx = 0; idx != word.size(); ++idx)
        {
            //if(word[idx] > 'A')
            if(!isalpha(word[idx]))
            {
                return string();//以空串进行替换
            }
        }

        return word;
    }

    //把结果插入到vector中
    void insert(const string &word)
    {
        if(word == string())
        {
            return;
        }

        size_t idx = 0;
        for(idx = 0; idx != _dict.size(); ++idx)
        {
            if(word == _dict[idx]._word)
            {
                ++_dict[idx]._frequency;
                break;//记得写上
            }
        }

        if(idx == _dict.size())
        {
            _dict.push_back(Record(word, 1));
        }
    }

private:
    vector<Record> _dict;
};

int main(int argc, char **argv)
{
    Dictionary dictionary(13000);
    cout << "before reading..." << endl;
    time_t beg = time(NULL);
    dictionary.read("The_Holy_Bible.txt");
    time_t end  = time(NULL);
    cout << "time : " << (end - beg) << "s" << endl;
    cout << "after reading..." << endl;
    dictionary.store("dictVector.dat");
    return 0;
}
```

## 文本查询

### 题目

```c++
该程序将读取用户指定的任意文本文件【当前目录下的china_daily.txt】，
然后允许用户从该文件中查找单词。查询的结果是该单词出现的次数，并列
出每次出现所在的行。如果某单词在同一行中多次出现，程序将只显示该行
一次。行号按升序显示。

要求：
a. 它必须允许用户指明要处理的文件名字。

b. 程序将存储该文件的内容，以便输出每个单词所在的原始行。
vector<string> lines;//O(1) 

c. 它必须将每一行分解为各个单词，并记录每个单词所在的所有行。 
在输出行号时，应保证以升序输出，并且不重复。 

map<string, set<int> > wordNumbers;
map<string, int> dict;

d. 对特定单词的查询将返回出现该单词的所有行的行号。

e. 输出某单词所在的行文本时，程序必须能根据给定的行号从输入
文件中获取相应的行。

示例：
使用提供的文件内容，然后查找单词 "element"。输出的前几行为:
---------------------------------------------
element occurs 125 times.
(line 62) element with a given key.
(line 64) second element with the same key.
(line 153) element |==| operator.
(line 250) the element type.
(line 398) corresponding element.
---------------------------------------------	

程序接口[可选]:
class TextQuery
{
public:
    //......
    void readFile(const string filename);
    //void query(const string & word);//查询和打印耦合在一起了
    QueryResult query(const string & word);
private:
    //......
    vector<string> _lines;//O(1) 
    map<string, set<int> > _wordNumbers;
    map<string, int> _dict;//
};

void print(ostream & os, const QueryResult &);

//程序测试用例
int main(int argc, char *argv[])
{
    string  queryWord("hello");

    TextQuery tq;
    tq.readFile("test.dat");
    tq.query(queryWord);			   
    return 0;
} 
```

### 答案

```c++
/* TextQuery.cc */
#include <iostream>
#include <string>
#include <fstream>
#include <sstream>
#include <vector>
#include <map>
#include <set>

using std::cout;
using std::endl;
using std::cin;
using std::ifstream;
using std::istringstream;
using std::vector;
using std::set;
using std::map;
using std::string;

class TextQuery
{
public:
    //构造函数先分配一定空间的大小
    TextQuery()
    {
        _file.reserve(100);
    }

    void readFile(const string &filename);
    void query(const string &word);

private:
    void preProceccLine(string &line);

private:
    //每次获取一行并存起来
    vector<string> _file;
    //单词以及词频
    map<string, int> _dict;
    //单词以及所在行号(注意：同一个单词在相同行出现，只记录一次)
    map<string, set<int>> _word2line;
};

void TextQuery::readFile(const string &filename)
{
    ifstream ifs(filename);
    if(!ifs)
    {
        cout << "ifstream open " << filename << " error!" << endl;
        return;
    }

    string line;
    size_t lineNumber = 0;
    while(getline(ifs, line))
    {
        //读一行，并将结果存在vector中(对单词处理前就存起来，存的是原始的)
        _file.push_back(line);

        //对读取行的单词进行处理
        preProceccLine(line);

        istringstream iss(line);
        string word;
        while(iss >> word)
        {
            //统计单词与词频
            ++_dict[word];

            //将单词所在的行记录下来
            _word2line[word].insert(lineNumber);
        }

        ++lineNumber;
    }

    ifs.close();
}

void TextQuery::query(const string &word)
{
    int count = _dict[word];//获取单词出现的次数
    cout << word << " occurs " << count << (count > 1 ? " times" : " time.") << endl;//打印单词次数

    auto lines = _word2line[word];//对同一个单词出现的行进行遍历，输出单词以及行号
    for(auto &number : lines)
    {
        cout << "    (line " << number + 1 << ") " << _file[number] << endl;
    }
}

void TextQuery::preProceccLine(string &line)
{
    for(auto &ch : line)
    {
        if(!isalpha(ch))//处理单词，如果不是字母就用空格代替abc? abc123 Hello
        {
            ch = ' ';
        }
        else if(isupper(ch))//如果是大写就转为小写,Hello
        {
            ch = tolower(ch);
        }
    }
}

int main(int argc, char **argv)
{
    TextQuery tq;
    tq.readFile("china_daily.txt");
    cout << " please input a query word: " << endl;
    string word;
    while(cin >> word)
    {
        tq.query(word);
    }

    return 0;
}
```





















