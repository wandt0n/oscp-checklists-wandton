C (i686)
```bash
i686-w64-mingw32-gcc in.c/in.cpp -o out.exe -lws2_32
```

C (x86_64)
```bash
x86_64-w64-mingw32-gcc in.c/in.cpp -o out.exe
```

> These commands should work as well for C++ code. If they don't:
	C++ (i686)
	```bash
	i686-w64-mingw32-g++ in.c/in.cpp -o out.exe -lws2_32
	```
	
	C++ (x86_64)
	```bash
	x86_64-w64-mingw32-g++ in.c/in.cpp -o out.exe
	```

**For DLLs** add a `--shared` to these commands

Get all compiler with `apt-cache search mingw-w64`

https://null-byte.wonderhowto.com/how-to/use-mingw-compile-windows-exploits-kali-linux-0179461/
