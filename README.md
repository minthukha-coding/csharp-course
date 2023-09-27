# C# Course

- Db Connectors
  - [Ado.Net](#adodotnet)
  - [Dapper](#Dapper)
  - [EF Core](#EFCore)


#### DataSet
> Dataset is the local copy of your database which exists in the local system and makes the application execute faster and reliable. DataSet works like a real database with an entire set of data which includes the constraints, relationship among tables, and so on. It will be found in the namespace “System. Data”.

#### DataTable
> DataTable is a fundamental part of the ADO.NET framework, which provides a way to work with structured tabular data in memory. It represents an in-memory, disconnected table of data, similar to a database table or spreadsheet. The DataTable class is part of the System.Data namespace.

#### DataRow
> DataRow is a fundamental component of the ADO.NET framework, specifically associated with the DataTable class. A DataRow represents a single row of data within a DataTable. Each row corresponds to a single record in the table, and it contains values for each of the columns defined in the DataTable. You can manipulate, access, and modify the data within a DataRow.

#### DataColumn
> DataColumn is another important component of the ADO.NET framework, specifically associated with the DataTable class. A DataColumn represents a single column within a DataTable. Each DataColumn has a name, data type, and other properties that define how data is stored and validated within that column.

In C#, ADO.NET provides a robust framework for performing CRUD (Create, Read, Update, Delete) operations on a relational database. Here's a summary of how you can use ADO.NET to implement CRUD operations:

1. **Create (INSERT)**:
   - To insert data into a database using ADO.NET, you typically start by creating a connection to the database using a connection string.
   - You then create an SQL INSERT command or a stored procedure that specifies the data to be inserted.
   - Create and open a `SqlConnection` to connect to the database.
   - Create a `SqlCommand` object with the INSERT command and associate it with the connection.
   - Set any parameters required for the INSERT operation, providing values for the new record.
   - Execute the command using `ExecuteNonQuery()` to insert the data into the database.

2. **Read (SELECT)**:
   - To retrieve data from a database using ADO.NET, again, create a connection to the database.
   - Create an SQL SELECT command or a stored procedure to fetch the desired data.
   - Create a `SqlCommand` object with the SELECT command and associate it with the connection.
   - Use a `SqlDataReader` or `SqlDataAdapter` to execute the command and retrieve the data.
   - Iterate through the results to read and process the retrieved data.

3. **Update (UPDATE)**:
   - To update existing data in a database using ADO.NET, establish a database connection.
   - Create an SQL UPDATE command or a stored procedure that specifies the data to be modified.
   - Create a `SqlCommand` object with the UPDATE command and associate it with the connection.
   - Set parameters for the UPDATE operation, including the new values and a condition to identify the record to update.
   - Execute the command using `ExecuteNonQuery()` to apply the changes to the database.

4. **Delete (DELETE)**:
   - For deleting data in a database, as usual, create a connection.
   - Construct an SQL DELETE command or use a stored procedure to specify the record(s) to be deleted.
   - Create a `SqlCommand` object with the DELETE command and associate it with the connection.
   - Set any necessary parameters, such as conditions for deletion.
   - Execute the command with `ExecuteNonQuery()` to remove the data from the database.

5. **Error Handling and Exception Handling**:
   - Proper error handling is crucial in ADO.NET to manage exceptions that may occur during database operations. Use try-catch blocks to handle exceptions and ensure that resources are properly disposed of in the event of an error.

6. **Connection Management and Resource Cleanup**:
   - Always close and dispose of database connections, commands, and readers when you're finished with them to release resources and prevent memory leaks.

7. **Parameterization**:
   - Use parameterized queries to protect against SQL injection attacks and to safely pass data between your C# code and the database.

8. **Transaction Management**:
   - For more complex operations involving multiple database commands, consider using transactions to ensure that a series of commands are executed as a single unit, either all succeeding or all failing.

ADO.NET provides a flexible and powerful way to interact with databases in C#, allowing you to implement CRUD operations efficiently and securely in your applications.

## AdoDotNet:
  
```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Linq;
using System.Reflection.Metadata;
using System.Text;
using System.Threading.Tasks;

namespace csharp_course
{
    public class AdoDotNetExample
    {
        private readonly SqlConnectionStringBuilder sqlConnectionStringBuilder = new SqlConnectionStringBuilder
        {
            DataSource = ".", //server name (local)
            InitialCatalog = "TestDb", //db name
            UserID = "sa", //user name
            Password = "sa@123" //password
        };

        public void Run()
        {
            Read();
            Create("title1", "author1", "content1");
            Edit(2);
            Update(2, "title2", "author2", "content2");
            Delete(1);
        }

        private void Read()
        {
            SqlConnection connection = new SqlConnection(sqlConnectionStringBuilder.ConnectionString);
            connection.Open();

            string query = "select * from tbl_blog";
            SqlCommand cmd = new SqlCommand(query, connection);
            SqlDataAdapter adapter = new SqlDataAdapter(cmd);
            DataTable dt = new DataTable();
            adapter.Fill(dt);

            connection.Close();

            foreach (DataRow dr in dt.Rows)
            {
                Console.WriteLine($"Blog Id => {dr["BlogId"].ToString()}");
                Console.WriteLine($"Blog Title => {dr["BlogTitle"].ToString()}");
                Console.WriteLine($"Blog Author => {dr["BlogAuthor"].ToString()}");
                Console.WriteLine($"Blog Content => {dr["BlogContent"].ToString()}");
            }
        }

        private void Edit(int id)
        {
            SqlConnection connection = new SqlConnection(sqlConnectionStringBuilder.ConnectionString);
            connection.Open();

            string query = "select * from tbl_blog where BlogId=@BlogId";
            SqlCommand cmd = new SqlCommand(query, connection);
            cmd.Parameters.AddWithValue("@BlogId", id);
            SqlDataAdapter adapter = new SqlDataAdapter(cmd);
            DataTable dt = new DataTable();
            adapter.Fill(dt);

            connection.Close();

            if(dt.Rows.Count == 0)
            {
                Console.WriteLine("No data found.");
                return;
            }

            DataRow dr = dt.Rows[0];
            Console.WriteLine($"Blog Id => {dr["BlogId"].ToString()}");
            Console.WriteLine($"Blog Title => {dr["BlogTitle"].ToString()}");
            Console.WriteLine($"Blog Author => {dr["BlogAuthor"].ToString()}");
            Console.WriteLine($"Blog Content => {dr["BlogContent"].ToString()}");
        }

        private void Create(string title, string author, string content)
        {
            SqlConnection connection = new SqlConnection(sqlConnectionStringBuilder.ConnectionString);
            connection.Open();

            string query = @"INSERT INTO [dbo].[Tbl_Blog]
           ([BlogTitle]
           ,[BlogAuthor]
           ,[BlogContent])
     VALUES
           (@BlogTitle
           ,@BlogAuthor
           ,@BlogContent)";
            SqlCommand cmd = new SqlCommand(query, connection);
            cmd.Parameters.AddWithValue("@BlogTitle", title);
            cmd.Parameters.AddWithValue("@BlogAuthor", author);
            cmd.Parameters.AddWithValue("@BlogContent", content);
            int result = cmd.ExecuteNonQuery();

            connection.Close();
            string message = result > 0 ? "Saving Successful.!" : "Saving Failed.";
            Console.WriteLine(message);
        }

        private void Update(int id, string title, string author, string content)
        {
            SqlConnection connection = new SqlConnection(sqlConnectionStringBuilder.ConnectionString);
            connection.Open();

            string query = @"UPDATE [dbo].[Tbl_Blog]
   SET [BlogTitle] = @BlogTitle
      ,[BlogAuthor] = @BlogAuthor
      ,[BlogContent] = @BlogContent
 WHERE BlogId = @BlogId";
            SqlCommand cmd = new SqlCommand(query, connection);
            cmd.Parameters.AddWithValue("@BlogId", id);
            cmd.Parameters.AddWithValue("@BlogTitle", title);
            cmd.Parameters.AddWithValue("@BlogAuthor", author);
            cmd.Parameters.AddWithValue("@BlogContent", content);
            int result = cmd.ExecuteNonQuery();

            connection.Close();
            string message = result > 0 ? "Updating Successful.!" : "Updating Failed.";
            Console.WriteLine(message);
        }

        private void Delete(int id)
        {
            SqlConnection connection = new SqlConnection(sqlConnectionStringBuilder.ConnectionString);
            connection.Open();

            string query = "delete from tbl_blog where blogId = @BlogId";
            SqlCommand cmd = new SqlCommand(query, connection);
            cmd.Parameters.AddWithValue("@BlogId", id);
            int result = cmd.ExecuteNonQuery();

            connection.Close();
            string message = result > 0 ? "Deleting Successful.!" : "Deleting Failed.";
            Console.WriteLine(message);
        }
    }
}
```
