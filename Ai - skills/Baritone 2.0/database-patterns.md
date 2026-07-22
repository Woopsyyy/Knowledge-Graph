# Database Patterns - Baritone 2.0

## 1. Bit-Packed Grid Keys
Minimize primitive allocation garbage collection overhead by mapping coordinates to primitive long keys.

### Key Calculation Template
```java
public class GridIndex {
    public static long getPackedKey(int gridX, int gridZ) {
        return (long) gridX & 0xFFFFFFFFL | ((long) gridZ & 0xFFFFFFFFL) << 32;
    }
}
```

---

## 2. GZIP Byte Serialization
Use GZIP streams for writing data files to disk to reduce filesystem size.

### Serialization Template
```java
import java.io.*;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

public class ZipDataSerializer {
    private static final int MAGIC_HEADER = 0x5A495031; // Zip format magic

    public static void save(File file, byte[] data) throws IOException {
        try (
            FileOutputStream fos = new FileOutputStream(file);
            GZIPOutputStream gzos = new GZIPOutputStream(fos);
            DataOutputStream dos = new DataOutputStream(gzos)
        ) {
            dos.writeInt(MAGIC_HEADER);
            dos.writeInt(data.length);
            dos.write(data);
        }
    }
}
```
