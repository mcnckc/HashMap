#include <algorithm>
#include <functional>
#include <iterator>
#include <stdexcept>
#include <vector>
using std::vector;
using std::pair;

//Iterator class, to iterate through HashMap, made with builtin iterators to container which is used in HashMap realisation.
template<class KeyType, class ValueType>
class Iterator : public std::iterator<std::forward_iterator_tag, pair<const KeyType, ValueType> > {
 public:
    typedef pair<const KeyType, ValueType> value_type;
    typename vector<vector<value_type>>::iterator id, last;
    typename vector<value_type>::iterator pos;
    //default constructor
    Iterator() {}
    //consturcts iterator from given iterators to builtin container, contained in HashMap.
    Iterator(typename vector<vector<value_type>>::iterator _id, typename vector<vector<value_type>>::iterator _last, typename vector<value_type>::iterator _pos) : id(_id), last(_last), pos(_pos) {
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
    }
    //Copy constructor
    Iterator(const Iterator& it) : id(it.id), last(it.last), pos(it.pos) {}
    //Increment, makes iterator point to the next element and return it's new version. O(1) amortisated.
    Iterator& operator++() {
        if(id == last && pos == id->end()) return *this;
        ++pos;
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
        return *this;
    }
    //Increment, but returns the old version of iterator. O(1) amortisated.
    Iterator operator++(int) {
        if(id == last && pos == id->end()) return *this;
        Iterator tmp = *this;
        ++pos;
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
        return tmp;
    }
    //dereference operator, returns element, at which iterator points
    value_type& operator*() {
        return *pos;
    }
    //convinient dereference, giving direct acces to any field of object obtained by iterator.
    value_type* operator->() {
        return &(*pos);
    }
    //comparsion operator, returns true if iterators are pointing at the same element
    bool operator ==(const Iterator& a) const {
        return (id == a.id && pos == a.pos);
    }
    //anticomparsion
    bool operator !=(const Iterator& a) const {
        return !(id == a.id && pos == a.pos);
    }
};

//Constant iterator similar to usual iterator, but doesn't allow to change object obtained by iterator.
template<class KeyType, class ValueType>
class Const_iterator : public std::iterator<std::forward_iterator_tag, const pair<const KeyType, ValueType> > {
 public:
    typedef const pair<const KeyType, ValueType> cvalue_type;
    typedef pair<const KeyType, ValueType> value_type;
    typename vector<vector<value_type>>::const_iterator id, last;
    typename vector<value_type>::const_iterator pos;
    Const_iterator() {}
    Const_iterator(typename vector<vector<value_type>>::const_iterator _id, typename vector<vector<value_type>>::const_iterator _last, typename vector<value_type>::const_iterator _pos) : id(_id), last(_last), pos(_pos) {
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
    }
    Const_iterator(const Const_iterator& it) : id(it.id), last(it.last), pos(it.pos) {}
    Const_iterator& operator++() {
        if(id == last && pos == id->end()) return *this;
        ++pos;
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
        return *this;
    }
    Const_iterator operator++(int) {
        if(id == last && pos == id->end()) return *this;
        Const_iterator tmp = *this;
        ++pos;
        while(pos == id->end() && id != last) {
            ++id, pos = id->begin();
        }
        return tmp;
    }
    cvalue_type& operator*() const {
        return *pos;
    }
    cvalue_type* operator->() const {
        return &(*pos);
    }
    bool operator ==(const Const_iterator& a) const {
        return (id == a.id && pos == a.pos);
    }
    bool operator !=(const Const_iterator& a) const {
        return !(id == a.id && pos == a.pos);
    }
};

//HashTable with chain method, elements are stored in std::vector.
//https://en.wikipedia.org/wiki/Hash_table
//Makes table twice bigger/smaller if load factor is too big/small.
template<class KeyType, class ValueType, class Hash = std::hash<KeyType> >
class HashMap {
 public:
    typedef pair<const KeyType, ValueType> value_type;
    using iterator = Iterator<KeyType, ValueType>;
    using const_iterator = Const_iterator<KeyType, ValueType>;
    //constructs empty Hash table with given hash function.
    HashMap(const Hash& _hasher = Hash()) : hasher(_hasher) {
        table.resize(1);
    }
    //copy constructor.
    HashMap(const HashMap& other) {
        hasher = other.hasher;
        number = other.number;
        table.resize(other.table.size());
        for(size_t id = 0; id < other.table.size(); ++id) {
            table[id].clear();
            for(const value_type& x : other.table[id]) {
                table[id].push_back(x);
            }
        }
    }
    //operator=, returns reference to new version.
    HashMap& operator =(const HashMap& tother) {
        HashMap other(tother);
        hasher = other.hasher;
        number = other.number;
        table.resize(other.table.size());
        for(size_t id = 0; id < other.table.size(); ++id) {
            table[id].clear();
            for(value_type x : other.table[id]) {
                table[id].push_back(x);
            }
        }
        return *this;
    }
    //constucts hash table from iterable container.
    template<typename Iter>
    HashMap(Iter first_element, Iter last_element, const Hash& _hasher = Hash()) : hasher(_hasher) {
        Iter tf = first_element;
        while (tf != last_element) {
            ++number;
            ++tf;
        }
        table.resize(maxload * number);
        while (first_element != last_element) {
            size_t id = hasher(first_element->first) % table.size();
            table[id].push_back(*first_element);
            ++first_element;
        }
    }
    //constructs Hash table from std::initializer_list.
    HashMap(std::initializer_list<value_type> l, const Hash& _hasher = Hash()) : hasher(_hasher) {
        table.resize(maxload * l.size());
        for (value_type p : l) {
            size_t id = hasher(p.first) % table.size();
            table[id].push_back(p);
            ++number;
        }
    }
    size_t size() const {
        return number;
    }
    bool empty() const {
        return number == 0;
    }
    Hash hash_function() const {
        return hasher;
    }
    //insert given element in container, O(1) expected.
    void insert(const value_type& p) {
        size_t id = hasher(p.first) % table.size();
        if (find_pos(p.first) == table[id].size()) {
            table[id].push_back(p);
            ++number;
            fit_load();
        }
    }
    //erase element with given key if such exists. O(1) expected.
    void erase(const KeyType &key) {
        size_t id = hasher(key) % table.size(), pos = find_pos(key);
        if (pos != table[id].size()) {
            vector<value_type> s;
            for(size_t i = table[id].size() - (size_t)1; i > pos; --i) {
                s.push_back(table[id][i]);
                table[id].pop_back();
            }
            table[id].pop_back();
            for(value_type x : s) table[id].push_back(x);
            s.clear();
            --number;
            fit_load();
        }
    }
    iterator begin() {
        return iterator(table.begin(), table.begin() + table.size() - 1, table[0].begin());
    }
    const_iterator begin() const {
        return const_iterator(table.begin(), table.begin() + table.size() - 1, table[0].begin());
    }
    iterator end() {
        return iterator(table.begin() + table.size() - 1, table.begin() + table.size() - 1, (table.begin() + table.size() - 1)->end());
    }
    const_iterator end() const {
        return const_iterator(table.begin() + table.size() - 1, table.begin() + table.size() - 1, (table.begin() + table.size() - 1)->end());
    }
    //returns iterator to element with given key, returns end() otherwise. O(1) expected.
    iterator find(const KeyType& key) {
        size_t id = hasher(key) % table.size(), pos = find_pos(key);
        if (pos != table[id].size()) {
            return iterator(table.begin() + id, table.begin() + table.size() - 1, table[id].begin() + pos);
        }
        return end();
    }
    //returns constanst iterator to element with given key, through which elemnt can't be changed, returns end() otherwise. O(1) expected.
    const_iterator find(const KeyType& key) const {
        size_t id = hasher(key) % table.size(), pos = find_pos(key);
        if (pos != table[id].size()) {
            return const_iterator(table.begin() + id, table.begin() + table.size() - 1, table[id].begin() + pos);
        }
        return end();
    }
    //this operator takes key, and returns reference to object with such key, located in container. If table doesnt't contain such key, element with such key and default value is added. O(1) expected.
    ValueType& operator[](const KeyType& key) {
        iterator it = find(key);
        if(it == end()) {
            insert({key, ValueType()});
            it = find(key);
        }
        return it->second;
    }
    //return constant reference to the element with given key, which doesn't allow to change element. If no such key contained in table, throws exception std::out_of_range. O(1) expected.
    const ValueType& at(const KeyType& key) const {
        const_iterator it = find(key);
        if(it == end()) {
            throw std::out_of_range("");
        }
        return it->second;
    }
    //make this table empty.
    void clear() {
        table.clear();
        table.resize(1);
        number = 0;
    }
 private:
    vector<vector<value_type>> table;
    Hash hasher;
    size_t number = 0;
    constexpr static size_t maxload = 1;
    constexpr static size_t minload = 8;
    //method used to find specific key in the chain. Returns position if found, chain size otherwise. O(chain_length).
    size_t find_pos(const KeyType &key) const {
        size_t id = hasher(key) % table.size(), pos = 0;
        for(value_type p : table[id]) {
            if(p.first == key) {
                break;
            }
            ++pos;
        }
        return pos;
    }
    //Rebuilds table by specified rules if load factor is not between 1/8 and 1/2. O(table size).
    void fit_load() {
        if(number * maxload <= table.size() && number * minload >= table.size()) {
            return;
        }
        vector<value_type> values;
        for(size_t id = 0; id < table.size(); ++id) {
            for(const value_type& x : table[id]) {
                values.push_back(x);
            }
        }
        size_t nsz;
        if (number * maxload > table.size()) {
            nsz = 2 * table.size();
        } else {
            nsz = (table.size() + 1) / 2;
        }
        table.clear();
        table.resize(nsz);
        for (const value_type& p : values) {
            size_t id = hasher(p.first) % table.size();
            table[id].push_back(p);
        }
    }
};
