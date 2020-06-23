# Csharp-connection-with-SQL-Server
This is an SQL server connection demo, using a desktop application C#-based. Written by : Salim Ali

1. Overview
      First think first, this project is just a demo that tests the connection between windows application (C# based)
    and a database stored in local server.

2. Procedure
      This small project is divided into two seperated parts. The Windows Desktop Application part, and the Database part.
    In the first part I used Visual Studio IDE to accomplish this part. In this environment I chose Windows Forms App (.Net Framework)
    C#-based and made the UI and its code as in attached files:
        UI:
        UI Code:
        Additional Classes:
    In other hand, I genrated a random personal info using this website: , first name, last name, email, and phone numbers. And stored this 
    information in a CSV file in my PC. Then I import them to a table (I called it [dbo].[People]) in local database using SQL Server Management System (SSMS). 
    
    2.1 Part I : Windows Desktop Application
    
    As said, I used VS IDE to design the UI: . The UI contains lables, buttons, textboxes, and listbox. the main UI does four missions, search
    in database.[People] by last name, delete a racord, show all records, and insert(add) new records. And as in  Additional Classes: .
    I create three additional classes [Person.cs, ConnectionToSQL.cs,and DataAccess.cs]. The Person.cs class contains properties that match
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
     with additional get property that return full Informaition. 
     
     The ConnectionToSQL.cs class has a static method which recieve a string parameter and make a connection using Configuration Maneger
     as follows:   
             
             using System.Configuration;
             public static class ConnectionToSQL
              {
                  public static string ConnectionValue(string nameee)
                  {
                      var op= ConfigurationManager.ConnectionStrings[nameee].ConnectionString;
                      return op;
                  }

              }
      Before talking about the last class, first I want to talk about the server connection, in App.config file I used this tag <xml> tag:
        <connectionStrings>
            <add name="salimConnection" connectionString="Server=.;         // Dot means that the connection is with a local server
            Database=SampleDB;Trusted_Connection=True;"
            providerName="System.Data.SqlClient" />
        </connectionStrings>
        
        this tag is making a connection between the project and SQL Server.
        
      Last but not least, the DataAccess.cs class. This class has five diffirent methods (GetPeople(),InsertData(),DeleteData(),
      Total(),and ShowAll()).This class work using Dapper NuGet package installed.
      GetPeople(string) it defined as a Person List and recieve a string parameter as follows:
     
          public List<Person> GetPeople(string lastName)
          {
              using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))

              {
                  return MyConnection.Query<Person>("dbo.People_GetByLastName @LName ", new { LName = lastName }).ToList();

              }

           }
      then return an SQL Query (stored procedure "dbo.People_GetByLastName @LName ") result.
      The scondnd method is InsertData() it defined as void method and recieve four parameters and using SQL query execution
      ([dbo].[People_Insertt] @FName, @LName ,@Email ,@Phone") to insert the four parameters as new record.

        public void InsertData(string firstName, string lastName, string email, string phone)
        {
          using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))
            {
                List<Person> people = new List<Person>();
                people.Add(new Person { FName = firstName, LName = lastName, Email = email, Phone = phone });
                MyConnection.Execute("[dbo].[People_Insertt] @FName, @LName ,@Email ,@Phone", people);
            }
        }
      The next is deletion method DeleteData() it recieve one parameter from txtLastName.Text and using the SQL query delete the record
        public void DeleteData(string lastNameD)
        {
            using (IDbConnection MyConnection = new System.Data.SqlClient.SqlConnection(ConnectionToSQL.ConnectionValue("salimConnection")))

            {
               MyConnection.Query<Person>($"Delete from People where LName='{lastNameD}'").ToList();
            }
        }

      The next methods are called and work together. ShowAll() recieves no parameters , using the stored procedure ($"dbo.showAllPeople")
      shows all dbo.People records on the screen (ListBox), and the other one, TotalR() shows the total of records.
        
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

        
    
    2.2 Part II : The Database
    
    
    
    
    When you click serach
    button (btnSearch) it behave:
    
        private void btnSearch_Click(object sender, EventArgs e)
        {
            DataAccess data  = new DataAccess();
            people = data.GetPeople(txtSearch.Text);
            UpdateBindings();
        }
     It instantiates a new object from DataAccess class and calls GetPeople(txtSearch.Text) method with the txtSearch.Text parameter and
