### Техника устная:

virtual память системные программирование устройство контейнеров map.

### Техника рефаторинга

\#ifndef LIBUTIL_LRU_CACHE_HPP__  
\#define LIBUTIL_LRU_CACHE_HPP__  

\#include <unordered_map>

namespace libutil  
{  

```Plain
template< class Tkey, class TItem >
class LastUsedCache
{
    struct record_t
    {
        TItem data;
        uint64_t gen;
    };

    typedef std::unordered_map< Tkey, record_t > cache_t;
    set<int> cache_min;
    cache_t cache_;
    size_t max_size_;
    uint64_t generation_;

public:
    LastUsedCache( size_t nelems )
    :   max_size_(nelems),
        generation_( 0 )
    {}

    bool find( Tkey key, TItem *item = NULL ) const
    {
        typename cache_t::const_iterator it = cache_.find( key );

        if (it == cache_.end())
            return false;
        else
        {
            if (item)
                *item = it->second.data;
            return true;
        }
    }
    //  5
    //  K V   K V
    // (1,1) (2,1) (2,1)
    // 1,1,1,1,1,1,1,2,3,4
    void insert( Tkey key, const TItem &data )
    {
        if (max_size_ == 0U)
        {
            return;
        }

        record_t rec{ data, ++ generation_ };

        auto result =
            cache_.insert( typename cache_t::value_type( key, rec ) );

        if (result.second != true) // key exists
        {
            result.first->second.gen = ++ generation_;
            return;
        }

        // Key is new
        if (cache_.size() > max_size_)
        {
            uint64_t min_gen = -1;
            for(auto & val : cache_ )
                min_gen = std::min( min_gen, val.second.gen );

            for( typename cache_t::iterator val_it = cache_.begin(); val_it != cache_.end(); ++ val_it )
                if( val_it->second.gen == min_gen )
                {
                    cache_.erase( val_it );
                    break;
                }
        }

    }

    size_t size() const
    {
        return cache_.size();
    }

    bool empty() const
    {
        return (size() == 0);
    }

    void clear()
    {
        cache_.clear();
    }
};
```

} // namespace libutil

\#endif // LIBUTIL_LRU_CACHE_HPP__

\#leetcode 132