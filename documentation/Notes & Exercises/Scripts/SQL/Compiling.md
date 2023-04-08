### SQL Compiling Code

To compile libraries for use with plugins in SQL:

1. Install the dependencies

   ```bash
   sudo apt install default-libmysqlclient-dev
   ```

2. Before compiling, ensure old object files are removed (these end with `.so`)

3. Adjust any includes in the makefile to fit your environment.

   - Ex.  If running mariaDB, then this:

     ```bash
     gcc -Wall -I/usr/include/mysql -I. -shared lib_mysqludf_sys.c -o $(LIBDIR)/lib_mysqludf_sys.so
     ```

     becomes this:

     ```bash
     gcc -Wall -I/usr/include/mariadb/server -I/usr/include/mariadb/ -I/usr/include/mariadb/server/private -I. -shared lib_mysqludf_sys.c -o lib_mysqludf_sys.so
     
     # Options
     -I	# These include the header file directories
     -shared	# Indicates this is a shared library & to generate a shared object file
     -o	# Output file
     ```

4. Compile the file

   ```bash
   make
   ```

   