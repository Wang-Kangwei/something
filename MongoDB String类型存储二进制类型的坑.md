结论：
MongoDB使用String类型存储二进制流（例如Protobuf序列化后的数据）时，使用Java驱动读取时可能会读取出和数据库中存储的内容不一致的数据.

主要在与Mongo驱动中，会将其转成UTF-8的字符串，若是存储的二进制流，则可能存储无法被UTF-8解析的字符，导致读取解析后的数据与存储的数据在二进制层面上不一致。
源码如下：
路径：bson-4.0.4-sources.jar!/org/bson/io/ByteBufferBsonInput.java
```
private String readString(final int size) {
        if (size == 2) {
            byte asciiByte = readByte();               // if only one byte in the string, it must be ascii.
            byte nullByte = readByte();                // read null terminator
            if (nullByte != 0) {
                throw new BsonSerializationException("Found a BSON string that is not null-terminated");
            }
            if (asciiByte < 0) {
                return UTF8_CHARSET.newDecoder().replacement();
            }
            return ONE_BYTE_ASCII_STRINGS[asciiByte];  // this will throw if asciiByte is negative
        } else {
            byte[] bytes = new byte[size - 1];
            readBytes(bytes); // 此处读取出来的bytes同mongodb中存储的内容是一样的
            byte nullByte = readByte();
            if (nullByte != 0) {
                throw new BsonSerializationException("Found a BSON string that is not null-terminated");
            }
            return new String(bytes, UTF8_CHARSET); // 在这将byte数组转String后，可能会导致与原始数据不一致
        }
    }
```
