# Csharp-connection-with-SQL-Server
This is an SQL server connection demo using a desktop application C#-based. Written by : Salim Ali

1. <bold>Overview</bold> <br>
      First things first, this project is just a demo that tests the connection between Windows application (C# based)
    and a database stored in a local server.

2. <strong>Procedure</strong> <br>
      This small project is divided into two seperate parts. The Windows Desktop Application part, and the Database part.
    In the first part I used Visual Studio IDE to accomplish it. In this environment I chose Windows Forms App (.Net Framework)
    C#-based and made the UI and its code as in attached files: <br>
        UI:<br>
        UI Code:<br>
        Additional Classes:<br>
    On other hand, I genrated a random personal info using this website: , first name, last name, email, and phone numbers. And stored these 
    data in a CSV file in my PC. Lastly, I imported them to a table (I called it [dbo].[People]) in a local database using SQL Server Management System (SSMS).<br> 
    
    2.1 Part I: Windows Desktop Application<br>
    
    As mentioned above, I used VS IDE to design the UI: <br> The UI contains lables, buttons, textboxes, and one listbox. The main UI does four missions, search in database.[People] by last name, delete a racord, show all records, and insert (add) new records. And as in  Additional Classes: <br>
    I created three additional classes [Person.cs, ConnectionToSQL.cs,and DataAccess.cs]. The Person.cs class contains properties that match
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
      Before talking about the last class, I want to talk about the server connection. In App.config file I used this tag <xml> <br>tag:
        <connectionStrings>
            <add name="salimConnection" connectionString="Server=.;         // Dot means that the connection is with a local server
            Database=SampleDB;Trusted_Connection=True;"
            providerName="System.Data.SqlClient" />
        </connectionStrings>
        
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
