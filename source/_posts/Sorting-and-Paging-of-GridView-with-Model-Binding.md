---
title: Sorting and Paging of GridView with Model Binding
date: 2017-12-21 12:51:45
tags:
---

Recently we got a bug from customer said that GridView sorting is not work well with paging enabled when using Model Binding. The "bug" behavior is different from when using an ObjectDataSource to populate the GridView, so bug was reported. 

Let us see the "bug" detail first:

Default.aspx
```
    <asp:GridView ID="userGridView" runat="server" AllowPaging="true" AllowSorting="true" AutoGenerateColumns="false"
        DataKeyNames="Id" SelectMethod="userGridView_GetData">
        <Columns>
            <asp:BoundField DataField="LastName" HeaderText="Last Name" SortExpression="LastName" />
            <asp:BoundField DataField="FirstName" HeaderText="First Name" SortExpression="FirstName" />
        </Columns>
    </asp:GridView>
```

Default.aspx.cs
```
namespace GridViewSort
{
    public class User
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public partial class _Default : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
        }

        public IEnumerable<User> GetUsers()
        {
            User[] users = new[]
            {
               new User { Id = 1, FirstName = "A", LastName = "One" },
               new User { Id = 2, FirstName = "B", LastName = "Two" },
               new User { Id = 3, FirstName = "C", LastName = "Three" },
               new User { Id = 4, FirstName = "D", LastName = "Four" },
               new User { Id = 5, FirstName = "E", LastName = "Five" },
               new User { Id = 6, FirstName = "F", LastName = "Six" }
            };

            return users.OrderByDescending(u => u.LastName);
        }

        public IQueryable<User> userGridView_GetData()
        {
            var users = GetUsers();

            return users.AsQueryable();
        }
    }
}
```
The ```GetUsers()``` returns the collection ordered by User LastName. When  set```AllowPaging ="false"```, GridView render as expected(rows ordered by LastName, even ```DataKeyNames="Id" ```), like this:
![image.png](http://upload-images.jianshu.io/upload_images/5123088-7281be8b9ce4ebad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

However, when set ```AllowPaging ="true"```, behavior changes, GridView rows not sorted by LastName anymore. like this: 
![image.png](http://upload-images.jianshu.io/upload_images/5123088-cfcf81b8f4b42a07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Then the "bug" was report, ```AllowPaging ="true"``` leads the behavior change. Customer cannot get an ordered GridView in the page initialize.

After our investigation, this is not a bug actually, ASP.NET model binding already provide a way to handle such scenario. The solution is update the ```SelectMethod``` to
```public IQueryable<User> userGridView_GetData(int maximumRows, int startRowIndex, out int totalRowCount, string sortByExpression)```
instead of 
```public IQueryable<User> userGridView_GetData()```

After some small change, the behavior must meet the customer's expected:
```
public IQueryable<User> userGridView_GetData(int maximumRows, int startRowIndex, out int totalRowCount, string sortByExpression)
        {
            var users = GetUsers();
            totalRowCount = users.Count();
            return users.Skip(startRowIndex).Take(maximumRows).AsQueryable();
        }
```
Then the result will be 
![image.png](http://upload-images.jianshu.io/upload_images/5123088-1865bcdc2495bf8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)