# Csharp-connection-with-SQL-Server 
<br>
This is an SQL server connection demo using a desktop application C#-based.<br>
<strong> Written by : Salim Ali</strong>

1. <strong>Overview</strong> <br>
      First things first, this project is just a demo that tests the connection between Windows application (C# based)
    and a database stored in a local server.

2. <strong>Procedure</strong> <br>
      This small project is divided into two seperate parts. The Windows Desktop Application part, and the Database part.
    In the first part I used Visual Studio IDE to accomplish it. In this environment I chose Windows Forms App (.Net Framework)
    C#-based and made the UI and its code as in attached files: <br>
        UI:<img>src="https://drive.google.com/file/d/1FtR4DfvSLT-jQwq8Q7DrR8Oak2S0wCYr/view"</img><br>
        UI Code:https://github.com/sal96ali/Csharp-connection-with-SQL-Server/blob/master/MainStructure<br>
        Additional Classes:https://github.com/sal96ali/Csharp-connection-with-SQL-Server/blob/master/AdditionalClasses<br>
    On other hand, I generated a random personal info using this website:http://www.randat.com/ to generate , first name, last name, email, and phone numbers. And stored these 
    data in a CSV file in my PC. Lastly, I imported them to a table that I called "[dbo].[People]" in a local database using SQL Server Management System (SSMS).<br> 
    
    2.1 Part I: Windows Desktop Application<br>
    
    As mentioned above, I used VS IDE to design the UI: <br> The UI contains lables, buttons, textboxes, and one listbox. The main UI does four missions, search in database.[People] by last name, delete a racord, show all records, and insert (add) new records. And as in  Additional Classes: https://github.com/sal96ali/Csharp-connection-with-SQL-Server/blob/master/AdditionalClasses<br>
    I created three additional classes [Person.cs, ConnectionToSQL.cs,and DataAccess.cs].<br> The Person.cs class contains properties that match
    with the dbo.People columns as follows:
    
            public class Person
            {
              public string FName { get; set; }
              public string LName { get; set; }
              public string Email { get; set; }
              public string Phone { get; set; }
              public string FullInfo
                {
                    get
                    {
                        return $"{FName} {LName}  Email:{Email}, Phone:{Phone}"; 
                    }

                }
            }
     with an additional get property that return full informaition. <br>
     
     The ConnectionToSQL.cs class has a static method, which recieves a string parameter and make a connection using Configuration Maneger
     as follows:   <br>
             
             using System.Configuration;
             public static class ConnectionToSQL
              {
                  public static string ConnectionValue(string nameee)
                  {
                      var op= ConfigurationManager.ConnectionStrings[nameee].ConnectionString;
                      return op;
                  }

              }
      Before talking about the last class, I want to talk about the server connection. In App.config file I used this xml tag: <br>
        &lt;connectionStrings &gt;
            &lt; add name="salimConnection" connectionString="Server=.;         &#47; &#47; Dot means that the connection is with a local server
            Database=SampleDB;Trusted_Connection=True;"
            providerName="System.Data.SqlClient" &#47; &gt;
        &lt;&#47; connectionStrings&gt;
        
        this tag is making a connection between the project and SQL Server.
        
      Last but not least, the DataAccess.cs class, that has five diffirent methods (GetPeople(),InsertData(),DeleteData(),
      Total(),and ShowAll()). This class works using Dapper NuGet package installed.
      GetPeople(string) that is defined as a Person List and recieves a string parameter as follows: <br>
     
          public List<Person> GetPeople(string lastName)
          {
              using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))

              {
                  return MyConnection.Query<Person>("dbo.People_GetByLastName @LName ", new { LName = lastName }).ToList();

              }

           }
           <br>
      then returns an SQL Query (stored procedure "dbo.People_GetByLastName @LName ") result.
      The second method is InsertData() that is defined as a void method and recieves four parameters and using SQL query execution
      ([dbo].[People_Insertt] @FName, @LName ,@Email ,@Phone") to insert the four parameters as a new record. <br>

        public void InsertData(string firstName, string lastName, string email, string phone)
        {
          using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))
            {
                List<Person> people = new List<Person>();
                people.Add(new Person { FName = firstName, LName = lastName, Email = email, Phone = phone });
                MyConnection.Execute("[dbo].[People_Insertt] @FName, @LName ,@Email ,@Phone", people);
            }
        }
      The next is deletion method DeleteData() that recieves one parameter from txtLastName.Text(TextBox), and using the SQL query to delete the record: <br>
        public void DeleteData(string lastNameD)
        {
            using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))

            {
               MyConnection.Query<Person>($"Delete from People where LName='{lastNameD}'").ToList();
            }
        }
<br>
      The next methods are being called and actually work together: ShowAll() that recieves no parameters, using the stored procedure ($"dbo.showAllPeople")
      that shows all dbo.People records on the screen (ListBox).TotalR(), on the other hand, shows the total of records. <br>
        
        public List<Person> ShowAll()
        {
            using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))
            {
                return MyConnection.Query<Person>($"dbo.showAllPeople").ToList();
            }
        }
        public string TotalR()
        {
            using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))
            {
               
               var output = MyConnection.ExecuteScalar("select count (*) from People");
                return output.ToString();
               
            }
        }

        
    <br>
    <strong>2.2 Part II : The Database</strong><br>
    
       
    As I mentioned earlier, I generated a random personal info using "random personal info website" to generate first name, last name, email, and phone numbers. And stored these data in a CSV file in my PC. Then, I imported them to a table which I called ([dbo].[People]) in a local database using SQL Server Management System (SSMS). Lastly, I made three stored procedures : [People_GetByLastName] ,[People_Insertt] ,and [showAllPeople].<br>
    
<strong>People_GetByLastName </strong> 
<br>
This stored procedure recieves one parameter, @LName nvarchar(50) and search in the database table by this parameter (last name) as follows:<br> 
    USE [SampleDB]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[People_GetByLastName]
	---- parameters 
      @LName nvarchar(50)
	 AS
BEGIN
	SET NOCOUNT ON;

      select *
	from dbo.People
	where LName = @LName
END
<br> 
<br>
    
<strong>People_Insertt </strong> 
The second one, recieves four parameters @FName nvarchar(50), @LName nvarchar(50), @Email nvarchar(50), @Phone nvarchar(50) rspectively, and
inserts these values into dbo.People table as follows: 
<br>
USE [SampleDB]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[People_Insertt]
@FName         nvarchar(50),
@LName             nvarchar(50),
@Email 	   nvarchar(50),
@Phone	   nvarchar(50)

AS
BEGIN
	
	SET NOCOUNT ON;

    -- Insert statements for procedure here

	insert into dbo.People(FName,LName,Email,Phone)
	values (@FName , @LName , @Email ,@Phone)
END
<br>
<strong>showAllPeople </strong><br>
The last one is showAllPeople, this one recieves no parameters and return an int value "count(all)" as follows :<br>

USE [SampleDB]
GO
/****** Object:  StoredProcedure [dbo].[showAllPeople]    Script Date: 23-Jun-20 23:09:45 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER PROCEDURE [dbo].[showAllPeople] 
	
AS
BEGIN
	
	SET NOCOUNT ON;

    SELECT * FROM People
	select COUNT (*) from People

END

    
    
<br> <strong>Results</strong><>
When app is opened the main UI is:
https://drive.google.com/file/d/1FtR4DfvSLT-jQwq8Q7DrR8Oak2S0wCYr/view?usp=drivesdk
With these buttons and textboxes we can access, search, delete,insert and show data from the database.
You can search for a specific person (by last name) by typing his last name in txtLastName textbox and click search button: <br>

<strong>Search button clicked with last name ="Jones"</strong> <br>
https://drive.google.com/file/d/1gWzbU6GZPNz0AGFLT22bSZafa3p8p34f/view?usp=drivesdk
<br>
You can delete a specific person by selecting his info on the listbox and click delete button:<br>
<strong>Delete button clicked with "Alford Jones" is selected</strong> <br>
https://drive.google.com/file/d/1jXYyrVuhpTXI6ZO_7GdjGo9ZiIdDiCZo/view?usp=drivesdk <br>
Note that there is an issue after deletion process.<br>
<strong>Search button clicked with last name is "Jones" (after deletion)</strong> <br>

https://drive.google.com/file/d/1nVUQEEYf7RTMy2VBdO8_-UcIJPc-qpsR/view?usp=drivesdk
<br>
After deletion, the problem is when I select a specific individual record, the delete method deletes all people with the same last names. This problem can be solved easily "discussed in the discussion paragraph".
<br>
To show all records click Show All button:<br>
<strong>Show All button clicked</strong> <br>
https://drive.google.com/file/d/15U_YTlzwnPFit5dfYknFQdWw2V1nZQxu/view?usp=drivesdk
<br>
Finally, to insert a new person, fill the information textboxes and click the Add button.<br>

<strong>button clicked with Salim Ali fake data</strong> <br>
https://drive.google.com/file/d/1htRBCkhny64hXoj2sTmzyElAGa9pD67E/view?usp=drivesdk
<br>
To show the new racord type the last last name and click search button: <br>
<strong> 
Search button clicked with Salim's last name "Ali" is typed</strong> <br>
https://drive.google.com/file/d/1-F8KxfFPjtEjXvKQTjrSpOp0MLdBKeqG/view?usp=drivesdk
<br>
<br>
<br>


    When you click serach
    button (btnSearch) it behave:
    
        private void btnSearch_Click(object sender, EventArgs e)
        {
            DataAccess data  = new DataAccess();
            people = data.GetPeople(txtSearch.Text);
            UpdateBindings();
        }
     It instantiates a new object from DataAccess class and calls GetPeople(txtSearch.Text) method with the txtSearch.Text parameter and
