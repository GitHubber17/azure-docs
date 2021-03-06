---
title: 'Connect to Azure Database for MySQL from C++ | Microsoft Docs'
description: This quickstart provides a C++ code sample you can use to connect and query data from Azure Database for MySQL.
services: mysql
author: seanli1988
ms.author: seal
manager: janders
editor: jasonwhowell
ms.service: mysql
ms.custom: mvc
ms.devlang: C++
ms.topic: quickstart
ms.date: 08/03/2017
---

# Azure Database for MySQL: Use Connector/C++ to connect and query data
This quickstart demonstrates how to connect to an Azure Database for MySQL using a C++ application. It shows how to use SQL statements to query, insert, update, and delete data in the database. The steps in this article assume that you are familiar with developing using C++, and that you are new to working with Azure Database for MySQL.

## Prerequisites
This quickstart uses the resources created in either of these guides as a starting point:
- [Create an Azure Database for MySQL server using Azure portal](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [Create an Azure Database for MySQL server using Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md)

You also need to:
- Install [.NET Framework](https://www.microsoft.com/net/download)
- Install [Visual Studio](https://www.visualstudio.com/downloads/)
- Install [MySQL Connector/C++](https://dev.mysql.com/downloads/connector/cpp/) 
- Install [Boost](http://www.boost.org/)

## Install Visual Studio and .NET
The steps in this section assume that you are familiar with developing using .NET.

### **Windows**
1. Install Visual Studio 2017 Community, which is a full featured, extensible, free IDE for creating modern applications for Android, iOS, Windows, as well as web & database applications and cloud services. You can install either the full .NET Framework or just .NET Core. The code snippets in the Quickstart work with either. If you already have Visual Studio installed on your machine, skip the next two steps.
   - Download the [Visual Studio 2017 installer](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15). 
   - Run the installer and follow the installation prompts to complete the installation.

### **Configure Visual Studio**
1. From Visual Studio, project property > configuration properties > C/C++ > linker > general > additional library directories, add the lib\opt directory (i.e.: C:\Program Files (x86)\MySQL\MySQL Connector C++ 1.1.9\lib\opt) of the c++ connector.
2. From Visual Studio, project property > configuration properties > C/C++ > general > additional include directories
   - Add include/ directory of c++ connector (i.e.: C:\Program Files (x86)\MySQL\MySQL Connector C++ 1.1.9\include\)
   - Add Boost library's root directory (i.e.: C:\boost_1_64_0\)
3. From Visual Studio, project property > configuration properties > C/C++ > linker > Input > Additional Dependencies, add mysqlcppconn.lib into the text field
4. Either copy mysqlcppconn.dll from the c++ connector library folder in step 3 to the same directory as the application executable or add it to the environment variable so your application can find it.

## Get connection information
Get the connection information needed to connect to the Azure Database for MySQL. You need the fully qualified server name and login credentials.

1. Log in to the [Azure portal](https://portal.azure.com/).
2. From the left-hand menu in Azure portal, click **All resources** and search for the server you have created, such as **myserver4demo**.
3. Click the server name.
4. Select the server's **Properties** page. Make a note of the **Server name** and **Server admin login name**.
 ![Azure Database for MySQL server name](./media/connect-cpp/1_server-properties-name-login.png)
5. If you forget your server login information, navigate to the **Overview** page to view the Server admin login name and, if necessary, reset the password.

## Connect, create table, and insert data
Use the following code to connect and load the data using **CREATE TABLE** and  **INSERT INTO** SQL statements. The code uses sql::Driver class with the connect() method to establish a connection to MySQL. Then the code uses method createStatement() and execute() to run the database commands. 

Replace the Host, DBName, User, and Password parameters with the values that you specified when you created the server and database. 

```c++
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/prepared_statement.h>
using namespace std;

int main()
{
	sql::Driver *driver;
	sql::Connection *con;
	sql::Statement *stmt;
	sql::PreparedStatement *pstmt;

	try
	{
		driver = get_driver_instance();
		//for demonstration only. never save password in the code!
		con = driver>connect("tcp://myserver4demo.mysql.database.azure.com:3306/quickstartdb", "myadmin@myserver4demo", "server_admin_password");
	}
	catch (sql::SQLException e)
	{
		cout << "Could not connect to database. Error message: " << e.what() << endl;
		system("pause");
		exit(1);
	}

	stmt = con>createStatement();
	stmt>execute("DROP TABLE IF EXISTS inventory");
	cout << "Finished dropping table (if existed)" << endl;
	stmt>execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);");
	cout << "Finished creating table" << endl;
	delete stmt;

	pstmt = con>prepareStatement("INSERT INTO inventory(name, quantity) VALUES(?,?)");
	pstmt>setString(1, "banana");
	pstmt>setInt(2, 150);
	pstmt>execute();
	cout << "One row inserted." << endl;

	pstmt>setString(1, "orange");
	pstmt>setInt(2, 154);
	pstmt>execute();
	cout << "One row inserted." << endl;

	pstmt>setString(1, "apple");
	pstmt>setInt(2, 100);
	pstmt>execute();
	cout << "One row inserted." << endl;
	
	delete pstmt;	
	delete con;
	system("pause");
	return 0;

```

## Read data

Use the following code to connect and read the data using a **SELECT** SQL statement. The code uses sql::Driver class with the connect() method to establish a connection to MySQL. Then the code uses method prepareStatement() and executeQuery() to run the select commands. Finally the code uses next() to advance to the records in the results. Then the code uses getInt() and getString() to parse the values in the record.

Replace the Host, DBName, User, and Password parameters with the values that you specified when you created the server and database. 

```csharp
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/prepared_statement.h>
using namespace std;

int main()
{
	sql::Driver *driver;
	sql::Connection *con;
	sql::PreparedStatement *pstmt;
	sql::ResultSet *result;

	try
	{
		driver = get_driver_instance();
		//for demonstration only. never save password in the code!
		con = driver>connect("tcp://myserver4demo.mysql.database.azure.com:3306/quickstartdb", "myadmin@myserver4demo", "server_admin_password");
	}
	catch (sql::SQLException e)
	{
		cout << "Could not connect to database. Error message: " << e.what() << endl;
		system("pause");
		exit(1);
	}	

//	select	
	pstmt = con>prepareStatement("SELECT * FROM inventory;");
	result = pstmt>executeQuery();	
	
	while (result>next())
		printf("Reading from table=(%d, %s, %d)\n", result>getInt(1), result>getString(2).c_str(), result>getInt(3));	
	
	delete result;
	delete pstmt;	
	delete con;
	system("pause");
	return 0;
}
```

## Update data
Use the following code to connect and read the data using a **UPDATE** SQL statement. The code uses sql::Driver class with the connect() method to establish a connection to MySQL. Then the code uses method prepareStatement() and executeQuery() to run the update commands. 

Replace the Host, DBName, User, and Password parameters with the values that you specified when you created the server and database. 

```csharp
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/prepared_statement.h>
using namespace std;

int main()
{
	sql::Driver *driver;
	sql::Connection *con;
	sql::PreparedStatement *pstmt;

	try
	{
		driver = get_driver_instance();
		//for demonstration only. never save password in the code!
		con = driver>connect("tcp://myserver4demo.mysql.database.azure.com:3306/quickstartdb", "myadmin@myserver4demo", "server_admin_password");
	}
	catch (sql::SQLException e)
	{
		cout << "Could not connect to database. Error message: " << e.what() << endl;
		system("pause");
		exit(1);
	}	

	//update
	pstmt = con>prepareStatement("UPDATE inventory SET quantity = ? WHERE name = ?");
	pstmt>setInt(1, 200);
	pstmt>setString(2, "banana");
	pstmt>executeQuery();
	printf("Row updated\n");
	
	delete con;
	delete pstmt;
	system("pause");
	return 0;
}
```


## Delete data
Use the following code to connect and read the data using a **DELETE** SQL statement. The code uses sql::Driver class with the connect() method to establish a connection to MySQL. Then the code uses method prepareStatement() and executeQuery() to run the delete commands.

Replace the Host, DBName, User, and Password parameters with the values that you specified when you created the server and database. 

```csharp
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/prepared_statement.h>
using namespace std;

int main()
{
	sql::Driver *driver;
	sql::Connection *con;
	sql::PreparedStatement *pstmt;
	sql::ResultSet *result;

	try
	{
		driver = get_driver_instance();
		//for demonstration only. never save password in the code!
		con = driver>connect("tcp://myserver4demo.mysql.database.azure.com:3306/quickstartdb", "myadmin@myserver4demo", "server_admin_password");
	}
	catch (sql::SQLException e)
	{
		cout << "Could not connect to database. Error message: " << e.what() << endl;
		system("pause");
		exit(1);
	}
		
	//delete
	pstmt = con>prepareStatement("DELETE FROM inventory WHERE name = ?");
	pstmt>setString(1, "orange");
	result = pstmt>executeQuery();
	printf("Row deleted\n");	
	
	delete pstmt;
	delete con;
	delete result;
	system("pause");
	return 0;
}
```

## Next steps
> [!div class="nextstepaction"]
> [Migrate your MySQL database to Azure Database for MySQL using dump and restore](concepts-migrate-dump-restore.md)
