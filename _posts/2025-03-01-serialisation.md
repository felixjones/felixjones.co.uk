---
layout: post
title: "Serialising Binary in C++23"
date: 2025-03-01
---

This is the first post in a series that documents the interesting things I encounter as I develop a Game Boy Advance game in 2025 (not yet announced).

---

I am British, so whilst my technical write-up will spell "serial*ize*" the British way (serial*ise*) any code samples will be in American English (ie: `serializer::deserialize`).

# Serialising Binary in C++23

I do not consider serialisation a difficult task, indeed there's the naïve tried-and-true technique of manually writing a binary writer and a binary reader for each type:

```c++
struct my_type {
    auto serialize(auto& archive) const {
        archive.write_integer(m_foo);
        archive.write_string(m_bar);
    }
    
    auto deserialize(auto& archive) const {
        m_foo = archive.read_integer();
        m_bar = archive.read_string();
    }
private:
    int m_foo{42};
    std::string m_bar{"Hello"};
};
```

But at work I have recently been writing some code for Minecraft Java Edition, and a feature in that codebase that I've always enjoyed is the codecs in the [DataFixerUpper library](https://github.com/Mojang/DataFixerUpper), specifically [see the syntax for serialising the `TestData` record](https://github.com/Mojang/DataFixerUpper/blob/d0f713b0719b5f1ad77506e6a019c41782b8bcf1/src/test/java/com/mojang/serialization/RoundtripTest.java#L73-L101):

```java
private record TestData(
    float a,
    double b,
    byte c,
    short d,
    int e,
    long f,
    boolean g,
    String h,
    List<String> i,
    Map<String, String> j,
    List<Pair<String, String>> k,
    DayData dayData
) {
    public static final Codec<TestData> CODEC = RecordCodecBuilder.create(i -> i.group(
        Codec.FLOAT.fieldOf("a").forGetter(d -> d.a),
        Codec.DOUBLE.fieldOf("b").forGetter(d -> d.b),
        Codec.BYTE.fieldOf("c").forGetter(d -> d.c),
        Codec.SHORT.fieldOf("d").forGetter(d -> d.d),
        Codec.INT.fieldOf("e").forGetter(d -> d.e),
        Codec.LONG.fieldOf("f").forGetter(d -> d.f),
        Codec.BOOL.fieldOf("g").forGetter(d -> d.g),
        Codec.STRING.fieldOf("h").forGetter(d -> d.h),
        Codec.STRING.listOf().fieldOf("i").forGetter(d -> d.i),
        Codec.unboundedMap(Codec.STRING, Codec.STRING).fieldOf("j").forGetter(d -> d.j),
        Codec.compoundList(Codec.STRING, Codec.STRING).fieldOf("k").forGetter(d -> d.k),
        DayData.CODEC.fieldOf("day_data").forGetter(d -> d.dayData)
    ).apply(i, TestData::new));
}
```

That `CODEC` static field is our serialiser, notice that there's no separate "serialize" and "deserialize" methods, just a definition of what fields there are, what the getters are, what constructor takes these arguments.

I really enjoy writing these `Codec` implementations, and the programmer zen of writing DFU Codecs is something I miss when I'm working on Bedrock in C++20.

---

Serialisation is needed for my Game Boy Advance side-project, and this gave me an opportunity to try implementing something closer to DFU's Codecs than what we traditionally see in the C++ serialiser landscape.

I don't need most of the features of DFU, I only need to write binary data and I don't even have to worry about versioning data for game updates considering this is a GBA project.

# All in on Constexpr

I've been using `constexpr` a lot for the GBA, and I consider it the primary thing that makes C++ a sensible choice for the GBA.

For a serialiser I believe `constexpr` is very important as we can give the compiler all the information about our types and it should, in theory, be able to optimise down to just calling the binary functions for encoding/decoding our types.

See what the above Java Codec looks like in my C++ interpretation:

```c++
struct test_data {
    float a;
    double b;
    unsigned char c;
    short d;
    int e;
    long long f;
    bool g;

    static constexpr auto codec = tuple_codec(
       float_codec.of_member(&test_data::a),
       double_codec.of_member(&test_data::b),
       unsigned_char_codec.of_member(&test_data::c),
       short_codec.of_member(&test_data::d),
       int_codec.of_member(&test_data::e),
       long_long_codec.of_member(&test_data::f),
       bool_codec.of_member(&test_data::g)
   ).apply<test_data>();
};
```

The `test_data::codec` static member is entirely built at compile-time, so the order of binary encoding/decoding functions to call for this type should be known to the compiler, letting it in theory optimise this all the way down to just the reading and writing of raw bytes.

---

In my current implementation I can see this:

<table><tr><td>C++</td><td>Arm (thumb)</td></tr><tr style="vertical-align:top"><td>

```c++
struct foo {
    int a;
    int b;
    int c;
    int d;

    static constexpr auto codec = tuple_codec(
       util::int_codec.of_member(&foo::a),
       util::int_codec.of_member(&foo::b),
       util::int_codec.of_member(&foo::c),
       util::int_codec.of_member(&foo::d)
   ).apply<foo>();
};

int main() {
    const std::string serialized = foo::codec.encode(foo{
        .a = 1,
        .b = 2,
        .c = 3,
        .d = 4
    });
    // ...
}
```

</td><td>

```arm
movs r3, #1           
movs r0, r5                            
add r1, sp, #24                    
str r3, [sp, #24]               
bl 0x8000780 <util::append_insert_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::operator=<std::array<char, 4u> >(std::array<char, 4u>&&)>
movs r3, #2                     
movs r0, r5                     
add r1, sp, #20                
str r3, [sp, #20]               
bl 0x8000780 <util::append_insert_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::operator=<std::array<char, 4u> >(std::array<char, 4u>&&)>
movs r3, #3                     
movs r0, r5                     
add r1, sp, #16                 
str r3, [sp, #16]               
bl 0x8000780 <util::append_insert_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::operator=<std::array<char, 4u> >(std::array<char, 4u>&&)>
movs r3, #4                     
movs r0, r5                     
add r1, sp, #12                 
str r3, [sp, #12]               
bl 0x8000780 <util::append_insert_iterator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >::operator=<std::array<char, 4u> >(std::array<char, 4u>&&)>
```

</td></tr></table>

Those four integer encodes are compiled into directly appending four `std::array<char, 4u>` chunks onto the end of a string.

I think this is remarkable, we're able to remove all the cost of the serialisation library down to just reading and writing bytes.

# Binary serialise...To a String?

In my little snippet above you'll notice that I serialised to a string:

```c++
const std::string serialized = foo::codec.encode(foo{ ... });
```

There's a feature that `std::string` has that `std::vector` does not: *Small String Optimisation*.

[Here's a link to Raymond Chen's article on the guts of `std::string` that covers this optimisation](https://devblogs.microsoft.com/oldnewthing/20230803-00/?p=108532), but the short of it is that if the data in a string is "small" then it won't bother with a heap allocation and will actually store the data within the string itself (which is probably on the stack).

Boost has [small_vector](https://live.boost.org/doc/libs/1_85_0/doc/html/boost/container/small_vector.html) which implements this for vector, but I'm not using Boost and I only need to serialise bytes, and when `sizeof(char) == 1` I might aswell use `std::string` instead of `std::vector<char>`.

Fundamental types will serialise to something within a small string, so this optimisation pays off for the GBA where heap allocations occur on the slow 256 KiB RAM.

# Encoding & Decoding Fundamental Types

The DataFixerUpper library has a set of primitive types that it has encoding operations for: defined in [DynamicOps.java](https://github.com/Mojang/DataFixerUpper/blob/master/src/main/java/com/mojang/serialization/DynamicOps.java#L22).

For C++ these would be our [fundamental types](https://en.cppreference.com/w/cpp/language/types).

Because our fundamental types are `constexpr` themselves, we can implement their encode/decode with `std::bit_cast` to and from an array of bytes:

```c++
template<typename T>
struct copy_encoder {
    template<std::convertible_to<T> O, AppendInserter Out>
    static constexpr void operator()(O&& obj, Out&& out) {
        out = std::bit_cast<std::array<char, sizeof(T)>>(std::forward<O>(obj));
    }
};

template<typename T>
struct copy_decoder {
    template<std::input_iterator In>
    static constexpr auto operator()(In&& in) {
        std::array<char, sizeof(T)> data;
        for (auto& c : data) {
            c = *in++;
        }
        return std::bit_cast<T>(data);
    }
};
```

Actually these copy encoders are good for anything that is `std::bit_cast` compatible (which is anything that is `std::is_trivially_copyable_v`), so the fundamental types can all use a `trivially_copyable_codec`:

```c++
template<typename T> requires std::is_trivially_copyable_v<T>
constexpr auto trivially_copyable_codec() {
    return codec(copy_encoder<T>{}, copy_decoder<T>{});
}

inline constexpr auto int_codec = trivially_copyable_codec<int>();
inline constexpr auto unsigned_int_codec = trivially_copyable_codec<unsigned int>();
inline constexpr auto short_codec = trivially_copyable_codec<short>();
inline constexpr auto unsigned_short_codec = trivially_copyable_codec<unsigned short>();
inline constexpr auto long_codec = trivially_copyable_codec<long>();
inline constexpr auto unsigned_long_codec = trivially_copyable_codec<unsigned long>();
inline constexpr auto long_long_codec = trivially_copyable_codec<long long>();
inline constexpr auto unsigned_long_long_codec = trivially_copyable_codec<unsigned long long>();
inline constexpr auto float_codec = trivially_copyable_codec<float>();
inline constexpr auto double_codec = trivially_copyable_codec<double>();
inline constexpr auto long_double_codec = trivially_copyable_codec<long double>();
inline constexpr auto char_codec = trivially_copyable_codec<char>();
inline constexpr auto signed_char_codec = trivially_copyable_codec<signed char>();
inline constexpr auto unsigned_char_codec = trivially_copyable_codec<unsigned char>();
inline constexpr auto wchar_codec = trivially_copyable_codec<wchar_t>();
inline constexpr auto char8_codec = trivially_copyable_codec<char8_t>();
inline constexpr auto char16_codec = trivially_copyable_codec<char16_t>();
inline constexpr auto char32_codec = trivially_copyable_codec<char32_t>();
inline constexpr auto bool_codec = trivially_copyable_codec<bool>();
```

If we have a type that is trivial then we have the option of a simple byte copy codec:

```c++
struct my_type {
    static consteval auto codec() { return util::trivially_copyable_codec<my_type>(); }

    int a;
    int b;
};
```

## The Arguably Fundamental Types

I'll never argue that `std::tuple` is a C++23 fundamental type, but things being tuple-like is quite fundamental in writing C++. Structs and Classes can be argued as being fancy tuples, so to support serialising these I have a generic `tuple_codec`.

```c++
template<typename... Codecs>
struct tuple_encoder {
    constexpr explicit tuple_encoder(Codecs... codecs) : m_encoder{ codecs.encoder()... } {}

    template<has_std_get T, AppendInserter Out>
    constexpr void operator()(T&& obj, Out&& out) const {
        [&]<std::size_t... I>(std::index_sequence<I...>) constexpr {
            (std::invoke(std::get<I>(m_encoder), std::get<I>(std::forward<T>(obj)), std::forward<Out>(out)), ...);
        }(std::make_index_sequence<std::tuple_size_v<decltype(m_encoder)>>{});
    }

    template<typename T, AppendInserter Out>
    constexpr void operator()(T&& obj, Out&& out) const {
        [&]<std::size_t... I>(std::index_sequence<I...>) constexpr {
            (std::invoke(std::get<I>(m_encoder), std::forward<T>(obj), std::forward<Out>(out)), ...);
        }(std::make_index_sequence<std::tuple_size_v<decltype(m_encoder)>>{});
    }
protected:
    std::tuple<encoder_type<Codecs>...> m_encoder;
};
```

Notice how the tuple encoder has an overload for using `std::get<>` versus forwarding the object reference to all of the tuple element encoders.

The latter overload is the mechanism that handles serialising member variables (the former is the true serialiser for tuple-like).

But what's the point of forwarding the same object to all the encoders? If the encoders were aware of the incoming object type, and also knew of a member function or member variabel of that object, then it can choose to encode that specific member of the object and nothing else. This is what the `codec<>::of_member` function is for.

---

The tuple decoder calls `std::make_tuple` using the results of the tuple element decoders.

```c++
template<typename... Codecs>
struct tuple_decoder {
    explicit constexpr tuple_decoder(Codecs... codecs) : m_decoder{ codecs.decoder()... } {}

    template<std::input_iterator In>
    constexpr auto operator()(In&& in) const {
        return [&]<std::size_t... I>(std::index_sequence<I...>) constexpr {
            return std::make_tuple(std::invoke(std::get<I>(m_decoder), std::forward<In>(in))...);
        }(std::make_index_sequence<std::tuple_size_v<decltype(m_decoder)>>{});
    }
protected:
    std::tuple<decoder_type<Codecs>...> m_decoder;
};
```

This certainly works for tuple-like, but what about our objects with serialised member variables?

That's where the `apply` comes into it, which adds decoders specific for member variables:

### Applying lambdas

```c++
template<typename Decoder, typename Ctor>
struct apply_lambda {
    constexpr apply_lambda(Decoder decoder, Ctor constructor) : m_decoder{ decoder }, m_constructor{ constructor } {}

    template<std::input_iterator In>
    constexpr auto operator()(In&& in) const {
        if constexpr (const auto result = std::invoke(m_decoder, std::forward<In>(in)); std::invocable<decltype(m_constructor), decltype(result)>) {
            return std::invoke(m_constructor, result); // For the case where we actually want a tuple as an argument
        } else {
            return std::apply(m_constructor, result); // For the case where we want to forward each tuple argument
        }
    }
protected:
    Decoder m_decoder;
    Ctor m_constructor;
};
```

The `apply_lambda` forwards the `std::make_tuple` result into the parameters of a given lambda, allowing for something like:

```c++
struct foo {
    int bar;
    int baz;

    static constexpr auto codec = tuple_codec(
        util::int_codec.of_member(&foo::bar),
        util::int_codec.of_member(&foo::baz)
    ).apply([](int bar, int baz) constexpr {
        // We could assert that our parameters are within constraints here
        return std::make_unique<foo>(bar, baz); // Always deserialises to a unique_ptr
    });
};
```

### Applying tuples

```c++
template<typename Decoder, typename T>
struct apply_tuple {
    constexpr explicit apply_tuple(Decoder decoder) : m_decoder{ decoder } {}

    template<std::input_iterator In>
    constexpr auto operator()(In&& in) const {
        return std::make_from_tuple<T>(std::invoke(m_decoder, std::forward<In>(in)));
    }
protected:
    Decoder m_decoder;
};
```

The `apply_tuple` forwards the result of `std::make_tuple` to [`std::make_from_tuple`](https://en.cppreference.com/w/cpp/utility/make_from_tuple), which will call whatever constructor for type `T` matches the given tuple arguments.

```c++
struct foo {
    int bar;
    int baz;

    static constexpr auto codec = tuple_codec(
        util::int_codec.of_member(&foo::bar),
        util::int_codec.of_member(&foo::baz)
    ).apply<foo>(); // Fowards to apply_tuple
};
```

### Applying ranges

```c++
template<typename Decoder, typename T>
struct apply_range {
    constexpr explicit apply_range(Decoder decoder) : m_decoder{ decoder } {}

    template<std::input_iterator In>
    constexpr auto operator()(In&& in) const {
        return std::ranges::to<T>(std::invoke(m_decoder, std::forward<In>(in)));
    }
protected:
    Decoder m_decoder;
};
```

Applying ranges only makes sense if we can support encoding ranges:

```c++
template<typename Codec, typename IndexCodec>
struct range_encoder {
    constexpr range_encoder(Codec codec, IndexCodec indexCodec) : m_encoder{ codec.encoder() }, m_indexEncoder{ indexCodec.encoder() } {}

    template<typename T, AppendInserter Out> requires std::ranges::range<T>
    constexpr void operator()(T&& range, Out&& out) const {
        std::invoke(m_indexEncoder, std::ranges::size(range), std::forward<Out>(out));
        std::ranges::for_each(range, [&]<typename O>(O&& obj) constexpr {
            std::invoke(m_encoder, std::forward<O>(obj), std::forward<Out>(out));
        });
    }
protected:
    encoder_type<Codec> m_encoder;
    encoder_type<IndexCodec> m_indexEncoder;
};
```

We can now serialise some standard containers:

```c++
static constexpr auto vector_of_int_codec = range_codec(util::int_codec).apply<std::vector<int>>();
static constexpr auto list_of_int_codec = range_codec(util::int_codec).apply<std::list<int>>();
static constexpr auto string_codec = range_codec(util::char_codec).apply<std::string>();
static constexpr auto key_value_codec = range_codec(tuple_codec(string_codec, string_codec));
```

### Applying ranges of tuples

The final step to this would be supporting `std::map`-like containers, which requires applying a range of tuples.

```c++
template<typename Decoder, typename T>
struct apply_range_of_tuple {
    constexpr explicit apply_range_of_tuple(Decoder decoder) : m_decoder{ decoder } {}

    template<std::input_iterator In>
    constexpr auto operator()(In&& in) const {
        return std::ranges::to<T>(std::invoke(m_decoder, std::forward<In>(in)));
    }
protected:
    Decoder m_decoder;
};
```

This applies to constructors such as `std::unordered_map`:

```c++
static constexpr auto map_codec = range_codec(
    tuple_codec(int_codec, int_codec)
).apply<std::unordered_map<int, int>>();
```

It was a very cool moment when writing the above worked first time, as I was expecting to be writing specialisations for `std::map`-like support.

# Now my compile times are long

These codecs add a good second or so to my compile time, and that could be unacceptable for larger projects.

## And the error messages are even longer

There is some very deeply nested templates at work here, so when a compile error does happen the resulting message is just impossible for a human to decipher. This has probably slowed down development of my binary serialiser the most.
