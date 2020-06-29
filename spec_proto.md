# Протокол

Протокол представляет собой смесь между текстовым и бинарным протоколом.
Путь регистра представлен в виде строки (ASCII), мета-информация и данные представленны в бинарном формате.

Сообщение представляет собой кортеж из 3-х элементов:

```none
(код_операции, путь_регистра, типизированные_данные)
```

Или 2-х, в случае операции чтения типизированные данные могут отсутствовать.

```none
(код_операции, путь_регистра)
```

## Типы данных

Протокол поддерживает следующие типы данных:

| Тип    | Описание                     | Длина/байты       |
|--------|------------------------------|-------------------|
| ()     | Пустой тип, аналог void      | 0                 |
| bool   | Булев тип                    | 1                 |
| u8-u32 | Целочисленные типы           | 1-4               |
| i8-i32 | Знаковые целочисленные       | 1-4               |
| str    | Строка переменной длины UTF-8| 0..MAX_PAYLOAD_SZ |
| [u8]   | Массив байт переменной длины | 0..MAX_PAYLOAD_SZ |

Тип данных кодируется полем `PAYLOAD_TY`.
Возможные коды типов данных см. в `enum TypeTag`.

## Формат

REQUEST:

Host -> Device

|  SIGN/PROTO_VER | PATH_SZ | PAYLOAD_SZ | REQ_CODE  | PAYLOAD_TY  |     PATH      |   PAYLOAD       |
|:---------------:|:-------:|:----------:|:---------:|:-----------:|:-------------:|:---------------:|
|     1 байт      | 1 байт  |   1 байт   |  1 байт   |    1 байт   |  PATH_SZ байт | PAYLOAD_SZ байт |

ANSWER:

Device -> Host

|  SIGN/PROTO_VER | PATH_SZ | PAYLOAD_SZ | ANS_CODE  | PAYLOAD_TY  |     PATH      |   PAYLOAD       |
|:---------------:|:-------:|:----------:|:---------:|:-----------:|:-------------:|:---------------:|
|     1 байт      | 1 байт  |   1 байт   |  1 байт   |    1 байт   |  PATH_SZ байт | PAYLOAD_SZ байт |

## Константы

```rust
# use std::mem::size_of;
#[repr(packed)]
pub struct Header {
    pub sign: u8,
    pub path_sz: u8,
    pub payload_sz: u8,
    pub code: u8,
    pub payload_ty: u8,
}

const MAX_MSG_SZ: usize = 512;
const HEADER_SZ: usize = size_of::<Header>();
const MAX_PATH_SZ: usize = MAX_MSG_SZ / 2 - HEADER_SZ;
// Размер ограничен максимальным значением u8, так что MAX_PAYLOAD_SZ: 2 -1  
const MAX_PAYLOAD_SZ: usize = MAX_MSG_SZ / 2 - 1;
// Сигнатура/Версия протокола
const SIGN: u8 = 0x8E;
```

```rust
#[repr(u8)]
pub enum RequestCode {
    READ = 0,
    WRITE = 1,
}
#[repr(u8)]
pub enum AnswerCode {
    OK_READ = 0,
    OK_WRITE = 1,
    ERR_LOCK = 2,
    ERR_BAD_PROTO = 3,
    ERR_BAD_FORMAT = 4,
    ERR_PATH = 5,
    ERR_TYPE = 6,
    ERR_USER = 7,
}
#[repr(u8)]
pub enum TypeTag {
    UNIT = 0,
    BOOL = 1,
    I32 = 2,
    I16 = 3,
    I8 = 4,
    U32 = 5,
    U16 = 6,
    U8 = 7,
    STR = 8,
    BYTES = 9,
}
```
