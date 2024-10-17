### ranges::range (C++20)
  
specifies that a type is a range, that is, it provides a begin iterator and an end sentinel
```c++
#include <ranges>
 
// A minimum range
struct SimpleRange
{
    int* begin();
    int* end();
};
static_assert(std::ranges::range<SimpleRange>);
 
// Not a range: no begin/end
struct NotRange
{
    int t {};
};
static_assert(!std::ranges::range<NotRange>);
 
// Not a range: begin does not return an input_or_output_iterator
struct NotRange2
{
    void* begin();
    int* end();
};
static_assert(!std::ranges::range<NotRange2>);
 
int main() {}
```
### ranges::borrowed_range (C++20)
  
specifies that a type is a range and iterators obtained from an expression of it can be safely returned without danger of dangling

```c++
#include <algorithm>
#include <array>
#include <cstddef>
#include <iostream>
#include <ranges>
#include <span>
#include <type_traits>
 
template<typename T, std::size_t N>
struct MyRange : std::array<T, N> {};
 
template<typename T, std::size_t N>
constexpr bool std::ranges::enable_borrowed_range<MyRange<T, N>> = false;
 
template<typename T, std::size_t N>
struct MyBorrowedRange : std::span<T, N> {};
 
template<typename T, std::size_t N>
constexpr bool std::ranges::enable_borrowed_range<MyBorrowedRange<T, N>> = true;
 
int main()
{
    static_assert(std::ranges::range<MyRange<int, 8>>);
    static_assert(std::ranges::borrowed_range<MyRange<int, 8>> == false);
    static_assert(std::ranges::range<MyBorrowedRange<int, 8>>);
    static_assert(std::ranges::borrowed_range<MyBorrowedRange<int, 8>> == true);
 
    auto getMyRangeByValue = []{ return MyRange<int, 4>{{1, 2, 42, 3}}; };
    auto dangling_iter = std::ranges::max_element(getMyRangeByValue());
    static_assert(std::is_same_v<std::ranges::dangling, decltype(dangling_iter)>);
    // *dangling_iter; // compilation error (i.e. dangling protection works.)
 
    auto my = MyRange<int, 4>{{1, 2, 42, 3}};
    auto valid_iter = std::ranges::max_element(my);
    std::cout << *valid_iter << ' '; // OK: 42
 
    auto getMyBorrowedRangeByValue = []
    {
        static int sa[4]{1, 2, 42, 3};
        return MyBorrowedRange<int, std::size(sa)>{sa};
    };
    auto valid_iter2 = std::ranges::max_element(getMyBorrowedRangeByValue());
    std::cout << *valid_iter2 << '\n'; // OK: 42
}
/*
42 42
*/
```

### ranges::sized_range (C++20)
  
 
specifies that a range knows its size in constant time
```c++
#include <forward_list>
#include <list>
#include <ranges>
 
static_assert
(
    std::ranges::sized_range<std::list<int>> and
    not std::ranges::sized_range<std::forward_list<int>>
);
 
int main() {}
```

### ranges::view (C++20)
  
 
specifies that a range is a view, that is, it has constant time copy/move/assignment
```c++
#include <ranges>
 
struct ArchetypalView : std::ranges::view_interface<ArchetypalView>
{
    int* begin();
    int* end();
};
 
static_assert(std::ranges::view<ArchetypalView>);
```

### ranges::input_range (C++20)
  
 
specifies a range whose iterator type satisfies input_iterator
```c++
```
### ranges::output_range (C++20)
  
 
specifies a range whose iterator type satisfies output_iterator
```c++
```
### ranges::forward_range (C++20)
  
 
specifies a range whose iterator type satisfies forward_iterator
```c++
#include <forward_list>
#include <queue>
#include <ranges>
#include <span>
#include <stack>
#include <tuple>
 
const char* str{"not a forward range"};
const char str2[] = "a forward range";
static_assert(
    std::ranges::forward_range<decltype("a forward range")> &&
    !std::ranges::forward_range<decltype(str)> &&
    std::ranges::forward_range<decltype(str2)> &&
    !std::ranges::forward_range<std::stack<char>> &&
    std::ranges::forward_range<std::forward_list<char>> &&
    !std::ranges::forward_range<std::tuple<std::forward_list<char>>> &&
    std::ranges::forward_range<std::span<char>> &&
    !std::ranges::forward_range<std::queue<char>> &&
"");
 
int main() {}
```
### ranges::bidirectional_range (C++20)
  
 
specifies a range whose iterator type satisfies bidirectional_iterator
```c++
#include <forward_list>
#include <list>
#include <ranges>
#include <set>
#include <unordered_set>
 
int main()
{
    static_assert(
            std::ranges::bidirectional_range<std::set<int>> and
        not std::ranges::bidirectional_range<std::unordered_set<int>> and
            std::ranges::bidirectional_range<std::list<int>> and
        not std::ranges::bidirectional_range<std::forward_list<int>>
    );
}
```

### ranges::random_access_range (C++20)
   
specifies a range whose iterator type satisfies random_access_iterator
```c++
#include <array>
#include <deque>
#include <list>
#include <ranges>
#include <set>
#include <valarray>
#include <vector>
 
template<typename T> concept RAR = std::ranges::random_access_range<T>;
 
int main()
{
    int a[4];
    static_assert(
            RAR<std::vector<int>> and
            RAR<std::vector<bool>> and
            RAR<std::deque<int>> and
            RAR<std::valarray<int>> and
            RAR<decltype(a)> and
        not RAR<std::list<int>> and
        not RAR<std::set<int>> and
            RAR<std::array<std::list<int>,42>>
    );
}
```
### ranges::contiguous_range (C++20)
   
specifies a range whose iterator type satisfies contiguous_iterator
```c++
#include <array>
#include <deque>
#include <list>
#include <ranges>
#include <set>
#include <valarray>
#include <vector>
 
template<typename T>
concept CR = std::ranges::contiguous_range<T>;
 
int main()
{
    int a[4];
    static_assert(
            CR<std::vector<int>> and
        not CR<std::vector<bool>> and
        not CR<std::deque<int>> and
            CR<std::valarray<int>> and
            CR<decltype(a)> and
        not CR<std::list<int>> and
        not CR<std::set<int>> and
            CR<std::array<std::list<int>,42>>
    );
}
```
### ranges::common_range (C++20)
  
 
specifies that a range has identical iterator and sentinel types
```c++
#include <ranges>
 
struct A
{
    char* begin();
    char* end();
};
static_assert( std::ranges::common_range<A> );
 
struct B
{
    char* begin();
    bool end();
};  // not a common_range: begin/end have different types
static_assert( not std::ranges::common_range<B> );
 
struct C
{
    char* begin();
};  // not a common_range, not even a range: has no end()
static_assert( not std::ranges::common_range<C> );
 
int main() {}
```
### ranges::viewable_range (C++20)
  
 
specifies the requirements for a range to be safely convertible to a view
```c++
#include <ranges>
#include <string>
#include <vector>
 
struct valid_result {};
struct invalid_result {};
 
template <typename T>
concept valid_viewable_range = std::same_as<T, valid_result>;
 
template <typename T>
concept invalid_viewable_range = std::same_as<T, invalid_result>;
 
auto test_viewable_range(std::ranges::viewable_range auto &&) -> valid_result;
auto test_viewable_range(auto&&) -> invalid_result;
 
int main()
{
    auto il = {1, 2, 3};
    int arr []{1, 2, 3};
    std::vector vec{1, 2, 3};
    std::ranges::ref_view r{arr};
    std::ranges::owning_view o{std::string("Hello")};
 
    static_assert(requires {
        { test_viewable_range(il) } -> valid_viewable_range;
        { test_viewable_range(std::move(il)) } -> invalid_viewable_range;
        { test_viewable_range(arr) } -> valid_viewable_range;
        { test_viewable_range(std::move(arr)) } -> invalid_viewable_range;
        { test_viewable_range(vec) } -> valid_viewable_range;
        { test_viewable_range(std::move(vec)) } -> valid_viewable_range;
        { test_viewable_range(r) } -> valid_viewable_range;
        { test_viewable_range(std::move(r)) } -> valid_viewable_range;
        { test_viewable_range(o) } -> invalid_viewable_range;
        { test_viewable_range(std::move(o)) } -> valid_viewable_range;
        { test_viewable_range(std::ranges::ref_view(o)) } -> valid_viewable_range;
    });
}
```
### ranges::constant_range (C++23)
   
specifies that a range has read-only elements
```c++
#include <ranges>
#include <span>
#include <string_view>
#include <vector>
 
// mechanisms for ensuring the parameter is a constant range
// 1) an overload set where the mutable one defers to the constant one
template<std::ranges::constant_range R>
void takes_any_range1(R&& r)
{
    // R is definitely a constant range
}
 
template<std::ranges::range R>
void takes_any_range1(R&& r)
{
    takes_any_range1(std::views::as_const(std::forward<R>(r)));
}
 
// 2) one function template that shadows its parameter
template<std::ranges::range R>
void takes_any_range2(R&& _r)
{
    auto r = std::views::as_const(std::forward<R>(_r));
 
    // r is definitely a constant range
    // never use _r again
}
 
// 3) one function template that recursively invokes itself
template<std::ranges::range R>
void takes_any_range3(R&& r)
{
    if constexpr (std::ranges::constant_range<R>)
    {
        // R is definitely a constant range
        // put implementation here
    }
    else
        takes_any_range3(std::views::as_const(std::forward<R>(r)));
}
 
static_assert
(
        std::ranges::constant_range<const std::vector<int>> and
    not std::ranges::constant_range<std::vector<int>> and
        std::ranges::constant_range<std::string_view> and
    not std::ranges::constant_range<std::span<int>> and
        std::ranges::constant_range<std::span<const int>> and
    not std::ranges::constant_range<const std::span<int>>
);
 
int main() {}
```
