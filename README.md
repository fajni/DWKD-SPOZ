# DWKD-SPOZ
Data Warehouse and Knowledge Discovery (Skladiste Podataka i Otkrivanje Znanja)

### Dataset

Updates:
<pre>
UPDATE SuperstoreSales
SET Customer_Segment = 'Small Business'
WHERE Customer_ID = 60
  AND Customer_Name = 'Sheila Warren';
</pre>

<hr/>

### DimProduct:

Create:
<pre>
CREATE TABLE DimProduct (
    Product_ID INT PRIMARY KEY IDENTITY(1,1),
    Product_Name NVARCHAR(100),
    Product_Department NVARCHAR(50),
    Product_Category NVARCHAR(50),
    Container NVARCHAR(50)
);
</pre>

Insert:
<pre>
INSERT INTO DimProduct(Product_Name, Product_Department, Product_Category, Container)
SELECT DISTINCT Product_Name, Product_Department, Product_Category, Container
FROM SuperstoreSales;
</pre>

### DimCustomer

Create:
<pre>
CREATE TABLE DimCustomer(
	Customer_ID smallint PRIMARY KEY IDENTITY(1,1),
	Customer_Name NVARCHAR(50),
	Customer_Segment NVARCHAR(50)
);
</pre>

Insert:
<pre>

</pre>

### DimOrder

Create:
<pre>
CREATE TABLE DimOrder(
	Order_Number INT PRIMARY KEY IDENTITY(1,1),
	Order_Priority NVARCHAR(50),
	Order_Date DATE,
	Ship_Date DATE,
	Ship_Mode NVARCHAR(50)
);
</pre>

Insert:
<pre>
INSERT INTO DimOrder (Order_Priority, Order_Date, Ship_Date, Ship_Mode)
SELECT DISTINCT Order_Priority, Order_Date, Ship_Date, Ship_Mode
FROM SuperstoreSales;
</pre>

### DimLocation

Create:
<pre>
CREATE TABLE DimLocation(
	Location_ID INT PRIMARY KEY IDENTITY(1,1),
	City NVARCHAR(50),
	Postal_Code int,
	State NVARCHAR(50),
	Country_Region NVARCHAR(50),
	Region NVARCHAR(50)
);
</pre>

Insert:
<pre>
INSERT INTO DimLocation(City, Postal_Code, State, Country_Region, Region)
SELECT DISTINCT City, Postal_Code, State, Country_Region, Region
FROM SuperstoreSales;
</pre>

### FactSales

Create:
<pre>
CREATE TABLE FactSales(
	Sale_ID int IDENTITY(1, 1),
	Product_ID int,
	Order_Number int,
	Customer_ID smallint,
	Location_ID int,

	CONSTRAINT FK_DimProduct FOREIGN KEY (Product_ID) REFERENCES DimProduct(Product_ID),
	CONSTRAINT FK_DimOrder FOREIGN KEY (Order_Number) REFERENCES DimOrder(Order_Number),
	CONSTRAINT FK_DimCustomer FOREIGN KEY (Customer_ID) REFERENCES DimCustomer(Customer_ID),
	CONSTRAINT FK_DimLocation FOREIGN KEY (Location_ID) REFERENCES DimLocation(Location_ID),
	CONSTRAINT PK_FactSales PRIMARY KEY (Sale_ID, Product_ID, Order_Number, Customer_ID, Location_ID),

	Order_Priority_Price tinyint,
	Discount decimal(18, 3),
	Unit_Price smallint,
	Order_Quantity smallint,
	Shipping_Cost smallint,
	Product_Base_Margin decimal(18, 3)
);
</pre>

Insert:
<pre>
INSERT INTO FactSales (Customer_ID, Location_ID, Order_Number, Product_ID, Order_Priority_Price, Discount, Unit_Price, Order_Quantity, Shipping_Cost, Product_Base_Margin)
SELECT c.Customer_ID, l.Location_ID, o.Order_Number, p.Product_ID, s.Order_Priority_Price, s.Discount, s.Unit_Price, s.Order_Quantity, s.Shipping_Cost, s.Product_Base_Margin
FROM SuperstoreSales s, DimCustomer c, DimLocation l, DimOrder o, DimProduct p
WHERE s.Customer_Name = c.Customer_Name AND s.Customer_Segment = c.Customer_Segment AND
s.City = l.City AND s.Postal_Code = l.Postal_Code AND s.State = l.State AND s.Country_Region = l.Country_Region AND
s.Order_Priority = o.Order_Priority AND s.Order_Date = o.Order_Date AND s.Ship_Date = o.Ship_Date AND s.Ship_Mode = o.Ship_Mode AND
s.Product_Name = p.Product_Name AND s.Product_Department = p.Product_Department AND s.Product_Category = p.Product_Category AND s.Container = p.Container;
</pre>