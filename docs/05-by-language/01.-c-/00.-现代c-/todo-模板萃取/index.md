# Todo 模板萃取<no value>

``` c++
    template <typename FUNC>
    void addListener(void *tag, FUNC &&func) {
        using stl_func = typename function_traits<typename std::remove_reference<FUNC>::type>::stl_function_type;
        Any listener;
        listener.set<stl_func>(std::forward<FUNC>(func));
        std::lock_guard<std::recursive_mutex> lck(_mtxListener);
        _mapListener.emplace(tag, listener);
    }

```