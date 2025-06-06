---
description: 'Руководство по интеграции библиотек Rust в ClickHouse'
sidebar_label: 'Библиотеки Rust'
slug: /development/integrating_rust_libraries
title: 'Интеграция библиотек Rust'
---


# Библиотеки Rust

Интеграция библиотеки Rust будет описана на примере интеграции хеш-функции BLAKE3.

Первым шагом интеграции является добавление библиотеки в папку /rust. Для этого необходимо создать пустой проект Rust и включить нужную библиотеку в Cargo.toml. Также необходимо настроить компиляцию новой библиотеки как статической, добавив `crate-type = ["staticlib"]` в Cargo.toml.

Далее нужно связать библиотеку с CMake, используя библиотеку Corrosion. Первый шаг — добавить папку с библиотекой в CMakeLists.txt внутри папки /rust. После этого следует добавить файл CMakeLists.txt в каталог библиотеки. В нем нужно вызвать функцию импорта Corrosion. Эти строки использовались для импорта BLAKE3:

```CMake
corrosion_import_crate(MANIFEST_PATH Cargo.toml NO_STD)

target_include_directories(_ch_rust_blake3 INTERFACE include)
add_library(ch_rust::blake3 ALIAS _ch_rust_blake3)
```

Таким образом, мы создадим правильную цель CMake с использованием Corrosion, а затем переименуем ее более удобным именем. Обратите внимание, что имя `_ch_rust_blake3` берется из Cargo.toml, где оно используется в качестве имени проекта (`name = "_ch_rust_blake3"`).

Поскольку типы данных Rust несовместимы с типами данных C/C++, мы будем использовать наш пустой библиотечный проект для создания методов-оберток для преобразования данных, полученных от C/C++, вызова методов библиотеки и обратного преобразования для выходных данных. Например, этот метод был написан для BLAKE3:

```rust
#[no_mangle]
pub unsafe extern "C" fn blake3_apply_shim(
    begin: *const c_char,
    _size: u32,
    out_char_data: *mut u8,
```
```rust
#[no_mangle]
pub unsafe extern "C" fn blake3_apply_shim(
    begin: *const c_char,
    _size: u32,
    out_char_data: *mut u8,
) -> *mut c_char {
    if begin.is_null() {
        let err_str = CString::new("input was a null pointer").unwrap();
        return err_str.into_raw();
    }
    let mut hasher = blake3::Hasher::new();
    let input_bytes = CStr::from_ptr(begin);
    let input_res = input_bytes.to_bytes();
    hasher.update(input_res);
    let mut reader = hasher.finalize_xof();
    reader.fill(std::slice::from_raw_parts_mut(out_char_data, blake3::OUT_LEN));
    std::ptr::null_mut()
}
```

Этот метод получает C-совместимую строку, ее размер и указатель на выходную строку в качестве аргументов. Затем он преобразует C-совместимые входы в типы, используемые фактическими методами библиотеки, и вызывает их. После этого он должен преобразовать выходные данные методов библиотеки обратно в C-совместимый тип. В данном случае библиотека поддерживала прямую запись в указатель с помощью метода fill(), поэтому преобразование не потребовалось. Главное здесь - создавать меньше методов, чтобы вам потребовалось меньше преобразований при каждом вызове метода и не создавать значительных накладных расходов.

Стоит отметить, что атрибут `#[no_mangle]` и `extern "C"` обязательны для всех таких методов. Без них нельзя будет выполнить правильную компиляцию, совместимую с C/C++. Более того, они необходимы для следующего шага интеграции.

После написания кода для методов-оберток нужно подготовить заголовочный файл для библиотеки. Это можно сделать вручную, либо использовать библиотеку cbindgen для автогенерации. В случае использования cbindgen вам нужно будет написать скрипт сборки build.rs и включить cbindgen как зависимость сборки.

Пример скрипта сборки, который может автоматически сгенерировать заголовочный файл:

```rust
    let crate_dir = env::var("CARGO_MANIFEST_DIR").unwrap();

    let package_name = env::var("CARGO_PKG_NAME").unwrap();
    let output_file = ("include/".to_owned() + &format!("{}.h", package_name)).to_string();

    match cbindgen::generate(&crate_dir) {
        Ok(header) => {
            header.write_to_file(&output_file);
        }
        Err(err) => {
            panic!("{}", err)
        }
    }
```

Также следует использовать атрибуты #[no_mangle] и `extern "C"` для каждого C-совместимого атрибута. Без этого библиотека может скомпилироваться некорректно, и cbindgen не запустит автогенерацию заголовка.

После всех этих шагов вы можете протестировать свою библиотеку в небольшом проекте, чтобы найти все проблемы с совместимостью или генерацией заголовков. Если возникают проблемы во время генерации заголовка, вы можете попробовать настроить его с помощью файла cbindgen.toml (шаблон можно найти здесь: [https://github.com/eqrion/cbindgen/blob/master/template.toml](https://github.com/eqrion/cbindgen/blob/master/template.toml)).

Стоит отметить проблему, которая возникла при интеграции BLAKE3:
MemorySanitizer может вызывать ложные срабатывания, так как он не может определить, инициализированы ли некоторые переменные в Rust или нет. Это было решено написанием метода с более явным определением для некоторых переменных, хотя эта реализация метода медленнее и используется только для исправления сборок с MemorySanitizer.
