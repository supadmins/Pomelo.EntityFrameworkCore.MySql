# Pomelo.EntityFrameworkCore.MySql

[![Travis build status](https://img.shields.io/travis/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql.svg?label=travis-ci&branch=master&style=flat-square)](https://travis-ci.org/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)
[![AppVeyor build status](https://img.shields.io/appveyor/ci/Kagamine/Pomelo-EntityFrameworkCore-MySql/master.svg?label=appveyor&style=flat-square)](https://ci.appveyor.com/project/Kagamine/pomelo-entityframeworkcore-mysql/branch/master) [![NuGet](https://img.shields.io/nuget/v/Pomelo.EntityFrameworkCore.MySql.svg?style=flat-square&label=nuget)](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/) [![Join the chat at https://gitter.im/PomeloFoundation/Home](https://badges.gitter.im/PomeloFoundation/Home.svg)](https://gitter.im/PomeloFoundation/Home?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

__Pomelo.EntityFrameworkCore.MySQL 1.0.0 RTM will be released on Aug 5th__

Pomelo.EntityFrameworkCore.MySql is an Entity Framework Core provider built on top of [Pomelo.Data.MySql](https://github.com/PomeloFoundation/Pomelo.Data.MySql). It makes you are able to use the Entity Framework Core ORM with MySQL.

## Getting Started

You are able to use MySQL in Entity Framework Core now, We have implemented MySQL Entity Framework Core interfaces. By using a few of lines to makes your project invoke Entity Framework Core with MySQL database. There is a console application sample for accessing MySQL database by using Entity Framework:

① Ensure `utf8` is your MySQL database default charset.
```sql
show variables like 'character_set_database';
```

If the result is not `utf8`, you should modify your `my.ini` or `my.cnf`.

② Put `Pomelo.EntityFrameworkCore.MySql` into your `project.json`
```json
{
  "version": "1.0.0-*",
  "buildOptions": {
    "emitEntryPoint": true
  },

  "tools": {
    "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final"
  },

  "dependencies": {
    "Microsoft.NETCore.App": {
      "type": "platform",
      "version": "1.0.0"
    },
    "Pomelo.EntityFrameworkCore.MySql": "1.0.0",
    "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final"
  },

  "frameworks": {
    "netcoreapp1.0": {
      "imports": "dnxcore50"
    }
  }
}
```

③ Implement some models, DbContext in `Program.cs`. Then overriding the OnConfiguring of DbContext to use MySQL database. Besides, you can define a JsonObject<T> field if you are using MySQL Server 5.7. Finally to invoking MySQL with EF Core in your Main() method.

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;

namespace MySqlTest
{
    public class User
    {
        public int UserId { get; set; }

        [MaxLength(64)]
        public string Name { get; set; }
    }

    public class Blog
    {
        public Guid Id { get; set; }

        [MaxLength(32)]
        public string Title { get; set; }

        [ForeignKey("User")]
        public int UserId { get; set; }

        public virtual User User { get; set; }

        public string Content { get; set; }

        public JsonObject<List<string>> Tags { get; set; } // Json storage (MySQL 5.7 only)
    }

    public class MyContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }

        public DbSet<User> Users { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            => optionsBuilder
                .UseMySql(@"Server=localhost;database=ef;uid=root;pwd=19931101;");
    }

    public class Program
    {
        public static void Main()
        {
            using (var context = new MyContext())
            {
                // Create database
                context.Database.EnsureCreated();

                // Init sample data
                var user = new User { Name = "Yuuko" };
                context.Add(user);
                var blog1 = new Blog {
                    Title = "Title #1",
                    UserId = user.UserId,
                    Tags = new List<string>() { "ASP.NET Core", "MySQL", "Pomelo" }
                };
                context.Add(blog1);
                var blog2 = new Blog
                {
                    Title = "Title #2",
                    UserId = user.UserId,
                    Tags = new List<string>() { "ASP.NET Core", "MySQL" }
                };
                context.Add(blog2);
                context.SaveChanges();

                // Changing and save json object #1
                blog1.Tags.Object.Clear();
                context.SaveChanges();

                // Changing and save json object #2
                blog1.Tags.Object.Add("Pomelo");
                context.SaveChanges();

                // Output data
                var ret = context.Blogs
                    .Where(x => x.Tags.Object.Contains("Pomelo"))
                    .ToList();
                foreach (var x in ret)
                {
                    Console.WriteLine($"{ x.Id } { x.Title }");
                    Console.Write("[Tags]: ");
                    foreach(var y in x.Tags.Object)
                        Console.Write(y + " ");
                    Console.WriteLine();
                }
            }
            Console.Read();
        }
    }
}
```

By viewing the following full project which is a single-user blog system and based on this library(MySQL for Entity Framework Core) to explorer more features: [View on GitHub](https://github.com/kagamine/yuukoblog-netcore-mysql).

## Schedule and Roadmap

Milestone | Release week
----------|-------------
1.0.0     | Aug 5th 2016
1.0.1     | Sep 2016

### 1.0.0 Feature complete

Support all Entity Framework Core operations, basic json field support.

### 1.0.1 Full json field support

Support translate JsonObject<T> linq queries to native sql.
 
``` c#
DB.Blogs.Where(x => x.Tags.Contains("MySQL")).ToList();
```
Translate to:
```sql
SELECT * FROM `Blogs` WHERE JSON_CONTAINS(`Tags`, '"MySQL"');
```

## Special supported features

- [MyCat cluster supported](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/issues/27)
- [MySQL 5.7 JSON field type](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/issues/14)

## Contribute

One of the easiest ways to contribute is to participate in discussions and discuss issues. You can also contribute by submitting pull requests with code changes.

## License

[MIT](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/blob/master/LICENSE)
