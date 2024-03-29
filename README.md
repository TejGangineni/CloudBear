Install Required Software:

Postman
Git
.Net SDK (.Net v7 is used for the demos).
Code editor or IDE (VS Code is used for all the demos)

Fast way - cloning the project
==============================

git clone https://github.com/TryCatchLearn/Skinet.git
cd Skinet
git reset --hard 80814b868221d2511a7a904bec8e5e536cc47b55

// Make sure project can build without errors

dotnet restore
dotnet build

// Create a new GitHub repository for this project and update the remote origin.

git remote set-url origin https://github.com/YOURGITHUBUSERBAME/name-of-repository.git

// Push the changes to your own GitHub repository

git push origin main


Slow way - manual approach to create the projects and get to the same point as end of section 2:
===============================================================================================

mkdir skinet
cd skinet
dotnet new sln
dotnet new webapi -o API
dotnet sln add API
dotnet new classlib -o Core
dotnet new classlib -o Infrastructure
dotnet sln add Core
dotnet sln add Infrastructure
cd API
dotnet add reference ../Infrastructure
cd ..
cd Infrastructure
dotnet add reference ../Core

Open the solution in your IDE of choice

Creating the required classes
=============================

Create a new folder in the Core project called Entities

Create new class called Product.cs with the following code:

namespace Core.Entitites
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}

Install Entity Framework Packages:

In the API project add a nuget package for:

Microsoft.EntityFrameworkCore.Design

In the Infrastructure project add nuget packages for:

Microsoft.EntityFrameworkCore.Sqlite

run the following command:

dotnet restore

Create a new Folder in the Infrastructure project called Data

Create a new Class in the Infrastructure project called StoreContext with the following code:

using Core.Entitites;
using Microsoft.EntityFrameworkCore;

namespace Infrastructure.Data
{
    public class StoreContext : DbContext
    {
        public StoreContext(DbContextOptions<StoreContext> options) : base(options)
        {
        }

        public DbSet<Product> Products { get; set; }
    }
}

Add a connection string in appsettings.development.json:

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Data source=skinet.db"
  }
}

Add the DbContext to the Program class (program class should look as follows):

using Infrastructure.Data;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace API
{
    public class Startup
    {
        private readonly IConfiguration _config;
        public Startup(IConfiguration config)
        {
            _config = config;
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            services.AddDbContext<StoreContext>(x =>
            {
                x.UseSqlite(_config.GetConnectionString("DefaultConnection"));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}


Adding migration and updating database:
======================================

Make sure you have the dotnet ef tools installed with the version of the sdk you are using:

dotnet tool install --global dotnet-ef

Run the following commmand:

dotnet ef migrations add InitialCreate -p Infrastructure -s API -o Data/Migrations

dotnet ef database update -p Infrastructure -s API

You should now have a skinet.db file.

Open the database and run the following query to add 3 entries into the DB:

-- SQLite
INSERT INTO `Products` (Id, Name)
VALUES (1, 'Product One'), (2, 'Product Two'), (3, 'Product Three')

Check the DB to make sure you can see 3 entries in there.

Adding the controller
=====================


Add a controller with the following code:


using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Core.Entities;
using Infrastructure.Data;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly StoreContext _context;
        public ProductsController(StoreContext context)
        {
            _context = context;
        }

        [HttpGet]
        public async Task<ActionResult<List<Product>>> GetProducts()
        {
            var products = await _context.Products.ToListAsync();

            return Ok(products);
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetProduct(int id)
        {
            return Ok(await _context.Products.FindAsync(id));
        }
    }
}


Add postman collection to postman.   Import skinet_postmancollection.json into your Postman and test the methods in Section 2 to make sure your controller can be reached. 

EF Migrations
==============================

dotnet ef migrations add InitialCreate -p Infrastructure -s API -o Data/Migrations
dotnet ef migrations remove -p Infrastructure -s API
dotnet ef database drop -p Infrastructure -s API    
dotnet ef database update -p Infrastructure -s API

GIT Commands
==============================

git add .
git commit -m migrationchanges
git push
git push origin main 
git remote add origin https://github.com/TejGangineni/CloudBear.git
git branch -m main 
git init   

DotNet Commands
==============================
dotnet watch --no-hot-reload
dotnet new list   
dotnet tool install --global dotnet-ef
dotnet new gitignore
dotnet restore
dotnet add reference ../Infrastructure
dotnet new sln
dotnet new webapi -o API
 dotnet sln add API    
 dotnet new classlib -o Core
