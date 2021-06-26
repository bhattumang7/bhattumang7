---
title: An example showing how to make an application less chatty over network.
tags: [Performance Optimization, SPE, software performance engineering, .Net Application Performance]
style: border 
color: light 
description: In this post, I go thru a situation where strategy from the book "Improving .Net Application Performance and Scalability" helped change an application that was not performing well.
---

Microsoft published a book with the title "Improving .Net Application Performance and Scalability" in the year 2005 (I was in high school then). It is an excellent book (considering that a lot of it is still relevant after so many years). It has several pieces of advice on tunings that we can make. I picked up an idea from there and applied it to a real-world problem. 

I was looking at an application that was taking a lot of time to do what it was supposed to do. The program was supposed to move a lot of data. I was trying to find out how to improve it. While running the SQL profiler, I saw that it invokes a lot of queries that do not take much time to execute in the database. The application also did not do much. A lot of time was spent on the network. In other words, the code I was looking at was too chatty with the database. 

Well, I knew that this is not a new problem being fixed for the first time. The book I mentioned earlier in chapter 12 describes "Improving ADO.Net Performance". In that chapter, they have a topic with the title "Reduce Round Trips". I decided to do the same thing. 


```csharp

List<MyRecord> myRecordList;

connection.Open(sourceStr);
command = new OleDbCommand("dbo.selectRecord", connection);
command.CommandType = CommandType.StoredProcedure;
command.Parameters.Add(new OleDbParameter("@ID", myId));
...//some more code here
reader = command.ExecuteReader();

while (reader.Read())
{
	record = new MyRecord();
	MyRecord.ID = reader.Get....
	... // some code to put right data in record
	myRecordList.Add(record);
}


connection1 = new OleDbConnection(targetStr);
connection1.Open();
foreach (MyRecord record in myRecordList)
{
	command = new OleDbCommand("dbo.insertStoreProcedure", connection1);
	command.CommandType = CommandType.StoredProcedure;
	command.Parameters.Add(new OleDbParameter("@ID", record.ID));
	.... // some code here
	rows = command.ExecuteNonQuery();
	command.Dispose();
}


```

For refactoring above code, I decided to use "SqlDataAdapter" and its batching capability. Sure, it was a lot more code; but it porformed a lot better. 

```csharp
connection = new SqlConnection(sourceStr);
connection.Open();
command = new SqlCommand()
{
	CommandType = CommandType.StoredProcedure,
	CommandText = "dbo.selectRecord",
	Connection = connection
};

connection1 = new SqlConnection(targetStr);
connection1.Open();

DataTable dataTable = new DataTable();

dataTable.Columns.Add("ID", typeof(int));

// create appropriate command
var writeCommand = new SqlCommand("dbo.insertStoreProcedure", connection1);
writeCommand.CommandType = CommandType.StoredProcedure;

// create the adapter
SqlDataAdapter adapter = new SqlDataAdapter();
adapter.ContinueUpdateOnError = false;
adapter.InsertCommand = writeCommand;
adapter.UpdateBatchSize = 500;

// add parameters to adapter
adapter.InsertCommand.Parameters.Add("@ID", SqlDbType....); // some code missing here

command.Parameters.Clear();

command.Parameters.Add(new SqlParameter("@Id", myId));
reader = command.ExecuteReader();

while (reader.Read())
{
	var newRow = dataTable.NewRow();
	newRow["ID"] = reader.Get...
	/// some more code here to populate newRow correctly.
	dataTable.Rows.Add(newRow);

	// if the rows are more than 499 flush the query to database
	if (dataTable.Rows.Count >= 499)
	{
		adapter.Update(dataTable);
		dataTable.Rows.Clear();
	}
}

// flush and other pending rows todatabase
if (dataTable.Rows.Count > 0)
{
	adapter.Update(dataTable);
	dataTable.Rows.Clear();
}


```
Once this change was in place, we saw significant improvement in how things performed (time taken).

Of course, the original code was a lot more complicated (with error-checking, etc.). To keep things easy to understand, making the example simplistic. 

Having an application that is too chatty is a recurring problem regardless of technology. Remember to take control of it when we spot it.
