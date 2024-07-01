# 哈希表的c++实现

本周借鉴邓俊辉的《数据结构（c++语言版）》，实现了一把简易的哈希表，算法导论第11章哈希表的笔记预计下周写好。

此哈希表实现使用了开放寻址策略（open addressing）中最基本的方法--线性试探法，以及通过使用位图来实现开放寻址策略的懒惰删除过程，详细的思路与算法解析得等到下周笔记写好才能看到了，代码放在下方：

BitMap.h

```c++
#ifndef BitMap_h
#define BitMap_h
#include <fstream>
#include <iostream>
#include <stdio.h>
#include <memory.h>
class Bitmap {
private:
    // Remember that the size of a char is a byte. 
    char* M; int N;
protected:
    void init ( int n ) { M = new char[N = (n + 7) / 8]; memset(M, 0, N); }
public:
    Bitmap (int n = 8) { init(n);}
    Bitmap (const char* file, int n = 8) //
    { 
        init(n);
        FILE* fp = fopen(file, "r");
        fread(M, sizeof(char), N, fp);
        fclose(fp);
    }
    ~Bitmap() { delete [] M; M = nullptr;}
    // Insert
    void DirectAddressInsert(int k) { expand(k); M[k >> 3] |= (0x80 >> ( k & 0x07)); }
    // Delete
    void DirectAddressDelete(int k) { expand(k); M[k >> 3] &= ~ (0x80 >> ( k & 0x07)); }
    // Test
    bool DirectAddressSearch(int k) { expand(k); return M[k >> 3] & (0x80 >> ( k & 0x07));}
    // expand memory size
    void expand(int k) {
        if (k < 8 * N) return;
        int oldN = N; char* oldM = M;
        init(2*k);
        memcpy(M, oldM, oldN);
        delete [] oldM; oldM = nullptr;
    }
    char* bits2string(int n) {
        expand(n-1);
        // 字符串所占空间，由上层调用者负责释放
        char* s = new char[n + 1]; s[n] = '\0';
        for (int i = 0; i < n; i++) {
            s[i] = DirectAddressSearch(i) ? '1' : '0';
        }
        return s;
    }

};


#endif /*BitMap_h*/
```

dictionary.h

```c++
#ifndef Dictionary_h
#define Dictionary_h

template <typename K, typename V> struct Entry {
    K key_;
    V val_; 
    Entry(K key = K(), V val = V()): key_(key), val_(val) {}
    Entry(Entry<K, V> const& e) : key_(e.key_), val_(e.val_) {}
    bool operator> (Entry<K, V> const& e) { return key_ > e.key_; }
    bool operator< (Entry<K, V> const& e) { return key_ < e.key_; }
    bool operator== (Entry<K, V> const& e) { return key_ == e.key_; }
    bool operator!= (Entry<K, V> const& e) { return key_ != e.key_; }
};
template <typename K, typename V> struct Dictionary {
    virtual int size() const = 0;
    virtual bool Insert (K key, V val) = 0;
    virtual V* Get(K key) = 0;
    virtual bool Remove(K key) = 0;
};

#endif /*Dictionary_h*/
```

Hashmap.h

```c++
#ifndef HashMap_h
#define HashMap_h
#include "BitMap.h"
#include "Dictionary.h"
#include <string>
#include <cstring>


template <typename K, typename V> class HashMap: public Dictionary<K,V> {
private:
    Entry<K, V>** ht;
    int M; // Bucket Number
    int N; //  key Number
    Bitmap* lazyRemoval;
    #define lazilyRemoved(x) (lazyRemoval->DirectAddressSearch(x))
    #define markAsRemoved(x) (lazyRemoval->DirectAddressInsert(x))
    // we may need mark
protected:
    int Probe4Hit (const K& key);
    int Probe4Free (const K& key);
    void Rehash();
public:
    HashMap(int c = 5);
    ~HashMap();
    int size() const { return N; }
    bool Insert (K key, V val);
    V* Get(K key);
    bool Remove(K key);
    size_t Hash(const K key);
};

#endif /*HashMap_h*/
```

HashMap.cpp

```c++
#include "HashMap.h"
static int primeNLT ( int c, int n, const char* file ) { //根据file文件中的记录，在[c, n)内取最小的素数
   Bitmap B ( file, n ); //file已经按位图格式，记录了n以内的所有素数，因此只要
   while ( c < n ) //从c开始，逐位地
      if ( B.DirectAddressSearch( c ) ) c++; //测试，即可
      else return c; //返回首个发现的素数
   return c; //若没有这样的素数，返回n（实用中不能如此简化处理）
}
static size_t hashCode(const char c) { return (size_t) c; }
static size_t hashCode(const int k) { return (size_t) k; }
static size_t hashCode(const long long i) { return (size_t) ((size_t)(i >> 32) + (int) i ); }
static size_t hashCode(const char s[]) {
    int h = 0;
    for (size_t n = strlen(s), i = 0; i < n; ++i) {
        // left cycle shift by 5 digits
        h = (h << 5) | (h >> 27); 
        h += (int) s[i];
    }
    return ( size_t ) h;
}
static size_t hashCode(const std::string& s) {
    int h = 0;
    for (size_t n = s.size(), i = 0; i < n; ++i) {
        // left cycle shift by 5 digits
        h = (h << 5) | (h >> 27); 
        h += (int) s[i];
    }
    return ( size_t ) h;
}
template <typename K, typename V> HashMap<K, V>::HashMap(int c) {
    M = primeNLT(c, 1048576, "prime-bitmap.txt");
    N = 0;
    ht = new Entry<K, V>*[M];
    memset(ht, 0, sizeof(Entry<K, V>*) *M);
    lazyRemoval = new Bitmap(M);
} 
template <typename K, typename V> HashMap<K, V>::~HashMap() {
    for (int i = 0; i < M; ++i) {
        if (ht[i]) delete ht[i]; ht[i] = nullptr;
    }
    delete [] ht; 
    ht = nullptr;
    delete lazyRemoval;
    lazyRemoval = nullptr;
} 
template <typename K, typename V> size_t HashMap<K, V>::Hash(const K key) {
    size_t hashcode = hashCode(key);
    return (4*hashcode + 3) % M;
}
// linear probing
template <typename K, typename V> int HashMap<K, V>::Probe4Hit(const K& key) {
    int rank = Hash(key);
    // case 1: the target key has not been found in a nonempty bucket
    // case 2: the bucket is empty and it has been marked as lazily removed
    while ((ht[rank] && (key != ht[rank]->key_)) || (!ht[rank] && lazilyRemoved(rank))) {
        rank = (rank + 1) % M;
    }
    return rank;
}
template <typename K, typename V> V* HashMap<K, V>::Get(K key) {
    int rank = Probe4Hit(key);
    return ht[rank] ? &(ht[rank]->val_) : nullptr;
}

template <typename K, typename V> bool HashMap<K, V>::Remove(K key) {
    int rank = Probe4Hit(key);
    if (!ht[rank]) return false;
    // release the entry element relating to this key word 
    delete ht[rank]; 
    ht[rank] = nullptr; 
    markAsRemoved(rank);
    --N;
    return true;
}
template <typename K, typename V> int HashMap<K, V>::Probe4Free(const K& key) {
    int rank = Hash(key);
    while ( ht[rank] ) {
        rank = (rank + 1) % M;
    }
    return rank;
}
template <typename K, typename V> bool HashMap<K, V>::Insert(K key, V val) {
    // 雷同元素不必重复插入
    if (ht[Probe4Hit(key)]) return false;
    int rank = Probe4Free(key);
    ht[rank] = new Entry<K, V>(key, val);
    ++N;
    if (N * 2 > M) {
        std::cout << "start rehashing.\n";
        Rehash();
    }
    return true;
}
template <typename K, typename V> void HashMap<K, V>::Rehash() {
    int old_capacity = M;
    Entry<K, V>** old_ht = ht;
    M = primeNLT(2*M, 1048576, "prime-bitmap.txt");
    N = 0; 
    ht = new Entry<K, V>*[M]; memset(ht, 0, sizeof(Entry<K, V>*) * M );
    delete lazyRemoval; 
    lazyRemoval = new Bitmap(M);
    for (int i = 0; i < old_capacity; ++i) {
        if (old_ht[i] ) {
            Insert(old_ht[i]->key_, old_ht[i]->val_);
        }
    }
    delete [] old_ht;
    old_ht = nullptr;
}

template class HashMap<int, std::string>;
template class HashMap<int, double>;
template class HashMap<std::string, std::string>;
```



main.cpp

```c++
// author: Claude Du

#include "HashMap.h"
#include <string>
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
#include <map>


int main()
{   

    HashMap<int, std::string> hmap;
    hmap.Insert(6, "We are the champions.");
    hmap.Insert(7, "Jaylen Brown, the FMVP of 2024");
    hmap.Insert(14, "What are they gonna say?");
    hmap.Insert(31, "I guess we will never know");
    hmap.Insert(13, "we were all on the same page in the locker room.");
    hmap.Insert(0, "Come on, u r Big Deuce!");
    std::cout << *(hmap.Get(0)) << "\n";
    std::cout << *(hmap.Get(14)) << "\n";
    std::cout << *(hmap.Get(31)) << "\n";
    hmap.Remove(0);
    if (hmap.Get(0) == nullptr) {
        std::cout << "removement operation works\n";
    }
}

```

通过命令行指令：

```shell
g++ hello.cpp HashMap.cpp -g
./a.exe
```

运行结果如下：

```shell
start rehashing.
start rehashing.
Come on, u r Big Deuce!
What are they gonna say?
I guess we will never know
removement operation works
```

HashTable.h

```c++
#ifndef HashTable_h
#define HashTable_h
#include "BitMap.h"
#include "Dictionary.h"
#include <string>
#include <cstring>
#include <vector>
#include <list>


template <typename K, typename V> class HashTable: public Dictionary<K,V> {
private:
    std::vector<std::list<Entry<K, V>>>* ht;
    int M; // Bucket Number
    int N; //  key Number
    typename std::list<Entry<K, V>>::iterator Find(int rank, K key);
protected:
    void Rehash();
public:
    HashTable(int c = 5);
    ~HashTable();
    int size() const { return N; }
    bool Insert (K key, V val);
    V* Get(K key);
    bool Remove(K key);
    size_t Hash(const K key);
};

#endif /*HashTable_h*/
```

HashTable.cpp

```c++
#include "HashTable.h"
static int primeNLT ( int c, int n, const char* file ) { //根据file文件中的记录，在[c, n)内取最小的素数
   Bitmap B ( file, n ); //file已经按位图格式，记录了n以内的所有素数，因此只要
   while ( c < n ) //从c开始，逐位地
      if ( B.DirectAddressSearch( c ) ) c++; //测试，即可
      else return c; //返回首个发现的素数
   return c; //若没有这样的素数，返回n（实用中不能如此简化处理）
}
static size_t hashCode(const char c) { return (size_t) c; }
static size_t hashCode(const int k) { return (size_t) k; }
static size_t hashCode(const long long i) { return (size_t) ((size_t)(i >> 32) + (int) i ); }
static size_t hashCode(const char s[]) {
    int h = 0;
    for (size_t n = strlen(s), i = 0; i < n; ++i) {
        // left cycle shift by 5 digits
        h = (h << 5) | (h >> 27); 
        h += (int) s[i];
    }
    return ( size_t ) h;
}
static size_t hashCode(const std::string& s) {
    int h = 0;
    for (size_t n = s.size(), i = 0; i < n; ++i) {
        // left cycle shift by 5 digits
        h = (h << 5) | (h >> 27); 
        h += (int) s[i];
    }
    return ( size_t ) h;
}
template <typename K, typename V> HashTable<K, V>::HashTable(int c) {
    M = primeNLT(c, 1048576, "prime-bitmap.txt");
    N = 0;
    ht = new std::vector<std::list<Entry<K, V>>>(M, std::list<Entry<K, V>>{});   
}
template <typename K, typename V> HashTable<K, V>::~HashTable() {
    for (auto& lst : (*ht)) {
        if (!lst.empty()) {
            lst.clear();
        }
    }
    ht->clear();
    delete ht;
    ht = nullptr;
    N = 0;
    M = 0;
}
template <typename K, typename V> size_t HashTable<K, V>::Hash(const K key) {
    size_t hashcode = hashCode(key);
    return (4*hashcode + 3) % M;
}
template <typename K, typename V> typename std::list<Entry<K, V>>::iterator HashTable<K, V>::Find(int rank, K key) {
    if ((*ht)[rank].empty()) return (*ht)[rank].end();
    for (auto ele = (*ht)[rank].begin(); ele != (*ht)[rank].end(); ++ele) {
        if (ele->key_ == key) return ele;
    }
    return (*ht)[rank].end();
}
template <typename K, typename V> bool HashTable<K, V>::Insert(K key, V val) {
    int rank = Hash(key);
    if (Find(rank, key) != (*ht)[rank].end()) return false;
    ++N;
    (*ht)[rank].push_back(Entry<K, V>(key, val));
    if (N * 2 > M) {
        std::cout << "start rehashing.\n";
        Rehash();
    }
    return true;
}
template <typename K, typename V> void HashTable<K, V>::Rehash() {
    int old_capacity = M;
    std::cout << "old capacity is " << old_capacity << std::endl;
    std::vector<std::list<Entry<K, V>>>* old_ht = ht;
    M = primeNLT(2*M, 1048576, "prime-bitmap.txt");
    std::cout << "new capacity is " << M << std::endl;

    ht = new std::vector<std::list<Entry<K, V>>>(M, std::list<Entry<K, V>>{}); 
    for (int i = 0; i < old_capacity; ++i) {
        if (!(*old_ht)[i].empty() ) {
            for (auto & ele : (*old_ht)[i]) {
                Insert(ele.key_, ele.val_);
            }
            (*old_ht)[i].clear();
        }
    }
    old_ht->clear();
    delete old_ht;
    old_ht = nullptr;
}
template <typename K, typename V> bool HashTable<K, V>::Remove(K key) {
    int rank = Hash(key);
    typename std::list<Entry<K, V>>::iterator toBedeleted = Find(rank, key);
    if (toBedeleted == (*ht)[rank].end()) return false;
    (*ht)[rank].erase(toBedeleted);
    --N;
    return true;

}
template <typename K, typename V> V* HashTable<K, V>::Get(K key) {
    int rank = Hash(key);
    typename std::list<Entry<K, V>>::iterator toBeUsed = Find(rank, key);
    if (toBeUsed == (*ht)[rank].end()) return nullptr;
    return &(toBeUsed->val_);
}
template class HashTable<int, std::string>;
template class HashTable<int, double>;
template class HashTable<std::string, std::string>;
```

