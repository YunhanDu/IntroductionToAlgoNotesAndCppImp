算法导论ch11散列表笔记与其c++实现

BitMap的实现

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
        memcpy_s(M, N, oldM, oldN);
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
#include <fstream>
#include <iostream>
#include <stdio.h>
#include <memory.h>
template <typename K, typename V> struct Entry {
    K key_;
    V val_; 
    Entry(K key = K(), V val = V()): key_(key), val_(val) {}
    Entry(Entry<K, V> const& e) : key_(e.key_), val_(e.val_) {}
    bool operater >(Entry<K, V> const& e) {
        e.key_ > key_;
    }
    bool operater <(Entry<K, V> const& e) {
        e.key_ < key_;
    }
    bool operater == (Entry<K, V> const& e) {
        e.key_ == key_;
    }
    bool operater != (Entry<K, V> const& e) {
        e.key_ != key_;
    }
}
template <typename K, typename V> struct Dictionary {
    virtual int size() const = 0;
    virtual bool insert (K key, V val) = 0;
    virtual V* get(K key) = 0;
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
int primeNLT ( int c, int n, const char* file ) { //根据file文件中的记录，在[c, n)内取最小的素数
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
static size_t hashCode(std::string& s) {
    int h = 0;
    for (size_t n = s.size(), i = 0; i < n; ++i) {
        // left cycle shift by 5 digits
        h = (h << 5) | (h >> 27); 
        h += (int) s[i];
    }
    return ( size_t ) h;
}
template <typename K, typename V> class HashMap<K, V>: public Dictionary<K,V> {
private:
    Entry<K, V>** ht;
    int M; // Bucket Number
    int N; //  key Number
    Bitmap* lazyRemoval;
    #define lazilyRemoved(x) (lazilyRemoved->DirectAddressSearch(x))
    #define markAsRemoved(x) (lazilyRemoved->DirectAddressInsert(x))
    // we may need mark
protected:
    int Probe4Hit (const K& key);
    int Probe4Free (const K& key);
    void Rehash();
public:
    Hashtable(int c = 5);
    ~Hashtable();
    int size() const { return N; }
    bool insert (K key, V val);
    V* get(K key);
    bool Remove(K key);
    size_t Hash(const K key);
};
template <typename K, typename V> HashMap<K, V>::HashMap(int c) {
    M = primeNLT(c, 1048576, "_input/prime-1048576-bitmap.txt");
    N = 0;
    ht = new Entry<K, V>*[M];
    memset(ht, 0, sizeof(Entry<K, V>*) *M);
    lazyRemoval = new Bitmap(M);
} 
template <typename K, typename V> HashMap<K, V>::~HashMap(int c) {
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
    return (4*hashCode + 3) % M;
}
// linear probing
template <typename K, typename V> size_t HashMap<K, V>::probe4Hit(const K& key) {
    int rank = Hash(key);
    // case 1: the target key has not been found in a nonempty bucket
    // case 2: the bucket is empty and it has been marked as lazily removed
    while ((ht[rank] && (key != ht[rank]->key_)) || (!ht[rank] && lazilyRemoved(rank))) {
        rank = (rank + 1) % M;
    }
    return rank;
}
template <typename K, typename V> V* HashMap<K, V>::get(K key) {
    int rank = probe4Hit(key);
    return ht[rank] ? &(ht[rank]->value_) : nullptr;
}

template <typename K, typename V> bool HashMap<K, V>::remove(K key) {
    int rank = probe4Hit(key);
    if (!ht[r]) return false;
    delete ht[r]; ht[r] = nullptr; 
    markAsRemoved(r);
    --N;
    return true;
}
#endif /*HashMap_h*/
```

main.cpp

```c++
// author: Claude Du
#include "TreeNode.h"
#include "AVLTree.h"
#include "BlackRedTree.h"
#include "BitMap.h"
#include <string>
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
#include <map>
#define SQUARE(a) ((a)*(a))
int primeNLT ( int c, int n, const char* file ) { //根据file文件中的记录，在[c, n)内取最小的素数
   Bitmap B ( file, n ); //file已经按位图格式，记录了n以内的所有素数，因此只要
   while ( c < n ) //从c开始，逐位地
      if ( B.DirectAddressSearch( c ) ) c++; //测试，即可
      else return c; //返回首个发现的素数
   return c; //若没有这样的素数，返回n（实用中不能如此简化处理）
}

int main()
{   
    int a = primeNLT(9, 1048576, "prime-bitmap.txt");
    std::cout << a << "\n";
}

```

