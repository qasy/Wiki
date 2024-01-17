PREFIX=/usr/local
CXX=c++
CXXFLAGS=

all: hello

hello: hello.cpp
	$(CXX) $(CXXFLAGS) -o hello hello.cpp

install: all
	mkdir -p $(PREFIX)/bin
	install hello $(PREFIX)/bin/

clean:
	rm hello

# uninstall: 
# 	rm $(PREFIX)/bin/hello