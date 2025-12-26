# Entity-Framework-Core
1️⃣ Introduction

ما هو Entity Framework Core؟
Entity Framework Core هو ORM (Object Relational Mapper) بيسمح لنا نتعامل مع قاعدة البيانات باستخدام كائنات C# بدل ما نكتب SQL Queries يدوي.

ليه نستخدم EF Core؟

يقلل كتابة SQL

يسهل التعامل مع البيانات

مناسب مع ASP.NET Core

يدعم Migrations

ORM يعني إيه؟
هو أداة بتربط بين:

Classes في الكود

Tables في قاعدة البيانات

بحيث كل Class يمثل Table وكل Property تمثل Column.

2️⃣ Add DbContext and Connection String

DbContext هو الكلاس الأساسي اللي بيتم من خلاله التواصل مع قاعدة البيانات.

الخطوات:

إنشاء كلاس يرث من DbContext

تعريف DbSet لكل Table

إضافة Connection String في appsettings.json

تسجيل DbContext داخل Program.cs
