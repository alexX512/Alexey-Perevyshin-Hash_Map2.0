#include <functional>
#include <list>
#include <vector>

template<class KeyType, class ValueType, class Hash = std::hash<KeyType>>
class HashMap {
private:
    const int initial_size = 4;
    typedef std::pair<KeyType, ValueType> pair_key_value;
    typedef std::pair<const KeyType, ValueType> pair_const_key_value;
    std::vector<pair_key_value> all_keys;
    std::vector<std::vector<int>> index_hashes;
    std::vector<std::pair<int, int>> keys_in_table;
    Hash hasher;

public:
    class iterator;
    class const_iterator;

private:
    iterator find_it(KeyType element) {
        size_t index = hasher(element) % index_hashes.size();
        for (size_t i : index_hashes[index]) {
            if (all_keys[i].first == element) {
                return iterator(i, this);
            }
        }
        return iterator(all_keys.size(), this);
    }

    const_iterator find_it(KeyType element) const {
        size_t index = hasher(element) % index_hashes.size();
        for (size_t i : index_hashes[index]) {
            if (all_keys[i].first == element) {
                return const_iterator(i, this);
            }
        }
        return const_iterator(all_keys.size(), this);
    }

    bool need_rebuild() {
        return all_keys.size() * initial_size > index_hashes.size();
    }

    void rebuild() {
        size_t last_size = index_hashes.size();
        auto tmp = all_keys;
        clear();
        index_hashes.resize(last_size * 2);
        for (const auto i : tmp) {
            insert(i);
        }
    }

public:
    class iterator {
    public:
        size_t id;
        HashMap* now;

        iterator(size_t _id = 0, HashMap* _now = nullptr) : id(_id), now(_now) {}

        iterator operator++(int) {
            iterator tmp = *this;
            ++id;
            return tmp;
        }

        iterator& operator++() {
            ++id;
            return *this;
        }

        iterator operator--(int) {
            iterator tmp = *this;
            --id;
            return tmp;
        }

        iterator& operator--() {
            --id;
            return *this;
        }

        pair_const_key_value& operator*() {
            return reinterpret_cast<pair_const_key_value&>(now->all_keys[id]);
        }

        pair_const_key_value* operator->() {
            return reinterpret_cast<pair_const_key_value*>(&now->all_keys[id]);
        }

        bool operator==(iterator other) const {
            return id == other.id;
        }

        bool operator!=(iterator other) const {
            return id != other.id;
        }
    };

    class const_iterator {
    public:
        size_t id;
        const HashMap* now;

        const_iterator(size_t _id = 0, const HashMap* _now = nullptr) : id(_id), now(_now) {}

        const_iterator operator++(int) {
            const_iterator tmp = *this;
            ++id;
            return tmp;
        }

        const_iterator& operator++() {
            ++id;
            return *this;
        }

        const_iterator operator--(int) {
            iterator tmp = *this;
            --id;
            return tmp;
        }

        const_iterator& operator--() {
            --id;
            return *this;
        }

        const pair_const_key_value& operator*() const {
            return reinterpret_cast<const pair_const_key_value&>(now->all_keys[id]);
        }

        const pair_const_key_value* operator->() const {
            return reinterpret_cast<const pair_const_key_value*>(&now->all_keys[id]);
        }

        bool operator==(const_iterator other) const {
            return id == other.id;
        }

        bool operator!=(const_iterator other) const {
            return id != other.id;
        }
    };

    HashMap(const Hash& _hasher = Hash()) : hasher(_hasher) {
        index_hashes.resize(initial_size);
    }

    template<typename Iter>
    HashMap(Iter _begin, Iter _end, const Hash& _hasher = Hash()) {
        index_hashes.resize(initial_size);
        hasher = _hasher;
        while (_begin != _end) {
            insert(*_begin);
            ++_begin;
        }
    }

    HashMap(std::initializer_list<pair_key_value> list, const Hash& _hasher = Hash()) {
        index_hashes.resize(initial_size);
        hasher = _hasher;
        for (auto i : list)
            insert(i);
    }

    size_t size() const {
        return all_keys.size();
    }

    bool empty() const {
        return all_keys.empty();
    }

    Hash hash_function() const {
        return hasher;
    }

    void insert(pair_key_value new_element) {
        if (need_rebuild()) {
            rebuild();
        }
        if (find_it(new_element.first).id == all_keys.size()) {
            size_t index = hasher(new_element.first) % index_hashes.size();
            keys_in_table.push_back(std::make_pair(index, index_hashes[index].size()));
            index_hashes[index].push_back(all_keys.size());
            all_keys.push_back(new_element);
        }
    }

    void erase(const KeyType delete_element) {
        size_t index = hasher(delete_element) % index_hashes.size();
        for (const int i : index_hashes[index]) {
            if (all_keys[i].first == delete_element) {
                std::swap(all_keys.back(), all_keys[i]);
                std::swap(keys_in_table.back(), keys_in_table[i]);
                std::pair<int, int> k2 = keys_in_table.back();
                std::pair<int, int> k1 = keys_in_table[i];

                std::swap(index_hashes[k1.first][k1.second], index_hashes[k2.first][k2.second]);
                std::swap(index_hashes[k2.first][k2.second], index_hashes[k2.first].back());
                keys_in_table[index_hashes[k2.first][k2.second]] = k2;
                index_hashes[k2.first].pop_back();
                all_keys.pop_back();
                keys_in_table.pop_back();
                break;
            }
        }
    }

    iterator begin() {
        return iterator(0, this);
    }

    iterator end() {
        return iterator(all_keys.size(), this);
    }

    const_iterator begin() const {
        return const_iterator(0, this);
    }

    const_iterator end() const {
        return const_iterator(all_keys.size(), this);
    }

    const_iterator find(KeyType element) const {
        return find_it(element);
    }

    iterator find(KeyType element) {
        return find_it(element);
    }

    ValueType& operator[] (KeyType element) {
        auto it = find_it(element);
        if (it.id == all_keys.size()) {
            insert(std::make_pair(element, ValueType()));
            return all_keys.back().second;
        }
        return it.now->all_keys[it.id].second;
    }

    const ValueType& at(KeyType element) const {
        auto it = find_it(element);
        if (it.id == all_keys.size()) {
            throw std::out_of_range("");
        }
        return it.now->all_keys[it.id].first;
    }

    void clear() {
        all_keys.clear();
        keys_in_table.clear();
        index_hashes.clear();
        index_hashes.resize(initial_size);
    }

};
