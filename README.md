# 1. Domain

The domain is **Motorbike Rental**, which includes:

1. Stakeholders such as renters, lenders, admins, maintenance staff, and accountants.
2. Processes like bike rentals, maintenance, financial tracking, and user management.
3. Business rules and constraints specific to renting and maintaining motorbikes.



## ** Step 1. Database Design**

### **Entities and Relationships**

1. **Renter** (One-to-Many with RentalTransaction):
   - A renter can create multiple rental transactions.
   
2. **Lender** (One-to-Many with Bike):
   - A lender can list multiple bikes for rent.
   
3. **Bike** (One-to-Many with RentalDetails, Many-to-One with Lender):
   - Each bike belongs to one lender but can be part of multiple rental transactions.
   
4. **RentalTransaction** (One-to-Many with RentalDetails, Many-to-One with Renter):
   - A transaction can include multiple bikes rented by one renter.
   
5. **RentalDetails** (Many-to-One with RentalTransaction, Many-to-One with Bike):
   - A junction table that records which bikes are rented as part of which transactions.
   
6. **MaintenanceStaff** (One-to-Many with MaintenanceRecord):
   - A staff member can perform maintenance on multiple bikes.

7. **MaintenanceRecord** (One-to-One with Bike, Many-to-One with MaintenanceStaff):
   - Each bike can have multiple maintenance records, but only one maintenance task is active at a time.

8. **Accountant** (One-to-Many with Payment):
   - The accountant oversees payments for multiple transactions.

9. **Payment** (One-to-One with RentalTransaction):
   - Each transaction has exactly one payment record.

---

### **Table Definitions**

1. **Renter**:
   - `RenterID` (Primary Key)
   - `Name`
   - `Email` (Unique)
   - `Phone`
   - `Address`
   - `AccountBalance`

---

2. **Lender**:
   - `LenderID` (Primary Key)
   - `Name`
   - `Email` (Unique)
   - `Phone`
   - `Address`

---

3. **Bike**:
   - `BikeID` (Primary Key)
   - `Model`
   - `Brand`
   - `Year`
   - `ConditionStatus` (`Available`, `Rented`, `UnderMaintenance`)
   - `DailyRate`
   - `LenderID` (Foreign Key)

---

4. **RentalTransaction**:
   - `RentalID` (Primary Key)
   - `RenterID` (Foreign Key)
   - `TotalCost` (Sum of costs in RentalDetails)
   - `RentalStartDate`
   - `RentalEndDate`
   - `PaymentStatus` (`Paid`, `Pending`)

---

5. **RentalDetails** (Junction Table for Many-to-Many):
   - `RentalDetailID` (Primary Key)
   - `RentalID` (Foreign Key referencing RentalTransaction)
   - `BikeID` (Foreign Key referencing Bike)
   - `RentalStartDate`
   - `RentalEndDate`
   - `Cost`

---

6. **MaintenanceStaff**:
   - `StaffID` (Primary Key)
   - `Name`
   - `Email`
   - `Phone`

---

7. **MaintenanceRecord**:
   - `MaintenanceID` (Primary Key)
   - `BikeID` (Foreign Key referencing Bike)
   - `StaffID` (Foreign Key referencing MaintenanceStaff)
   - `MaintenanceDate`
   - `IssueDescription`
   - `ResolutionDetails`
   - `Cost`

---

8. **Accountant**:
   - `AccountantID` (Primary Key)
   - `Name`
   - `Email`
   - `Phone`

---

9. **Payment**:
   - `PaymentID` (Primary Key)
   - `RentalID` (Foreign Key referencing RentalTransaction)
   - `Amount`
   - `PaymentDate`
   - `Status` (`Paid`, `Pending`)

---

### **Relationships:**

| Relationship                   | Type                  |
|--------------------------------|-----------------------|
| Renter ↔ RentalTransaction     | One-to-Many           |
| Lender ↔ Bike                  | One-to-Many           |
| Bike ↔ RentalDetails           | One-to-Many           |
| RentalTransaction ↔ RentalDetails | One-to-Many           |
| MaintenanceStaff ↔ MaintenanceRecord | One-to-Many           |
| MaintenanceRecord ↔ Bike       | One-to-One (per task) |
| Accountant ↔ Payment           | One-to-Many           |
| RentalTransaction ↔ Payment    | One-to-One            |

---

### Example Scenario:

1. **Renter Alice rents 2 bikes:**
   - **RentalTransaction**:
     - `RentalID`: 1001
     - `RenterID`: 2001 (Alice)
     - `TotalCost`: $250
     - `PaymentStatus`: Pending
   - **RentalDetails**:
     - RentalDetail 1: BikeID = 3001, Cost = $100
     - RentalDetail 2: BikeID = 3002, Cost = $150

2. **Maintenance Record for a Bike:**
   - **BikeID**: 3001 is sent for maintenance.
   - **MaintenanceRecord**:
     - MaintenanceID = 4001
     - BikeID = 3001
     - StaffID = 5001
     - IssueDescription = "Brake malfunction."
     - Cost = $50.

---
### **Step 2: Business Logic**

1. **Single RentalTransaction for Multiple Bikes**:
   - Each `RentalTransaction` is created for one renter but can include multiple bikes via the `RentalDetails` table.
   - The system calculates the `TotalCost` by summing up the `Cost` from the `RentalDetails` table.

2. **Bike Availability**:
   - A bike can only be rented if its `ConditionStatus` is "Available."
   - Upon renting, the status is updated to "Rented."

3. **Payment Processing**:
   - Payments are linked to the `RentalTransaction` through the `Payment` table.
   - Payments can be "Pending" or "Paid," and the system blocks new rentals for renters with pending payments.

4. **Maintenance Workflow**:
   - When a bike requires maintenance, the `ConditionStatus` is updated to "UnderMaintenance."
   - A maintenance record is created for each repair, and the bike becomes "Available" once resolved.

5. **Role-Based Access**:
   - **Renter**: Can view bikes, create rentals, and manage their account.
   - **Lender**: Can manage their bikes and view rental details.
   - **Admin**: Full control over the system, including users and transactions.
   - **Maintenance Staff**: Limited to viewing and updating maintenance records.
   - **Accountant**: Focuses on payment records and financial reports.

6. **Constraints**:
   - A bike cannot belong to multiple active rentals.
   - Overlapping rental periods for the same bike are disallowed.

---

### **Step 3: Schema Definitions**

#### **SQL Schema:**

```sql
-- Renter Table
CREATE TABLE Renter (
    RenterID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15),
    Address TEXT,
    AccountBalance DECIMAL(10, 2)
);

-- Lender Table
CREATE TABLE Lender (
    LenderID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15),
    Address TEXT
);

-- Admin Table
CREATE TABLE Admin (
    AdminID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15)
);

-- Bike Table
CREATE TABLE Bike (
    BikeID INT PRIMARY KEY AUTO_INCREMENT,
    Model VARCHAR(100) NOT NULL,
    Brand VARCHAR(100),
    Year INT,
    ConditionStatus ENUM('Available', 'Rented', 'UnderMaintenance') DEFAULT 'Available',
    DailyRate DECIMAL(10, 2) NOT NULL,
    LenderID INT,
    FOREIGN KEY (LenderID) REFERENCES Lender(LenderID)
);

-- RentalTransaction Table
CREATE TABLE RentalTransaction (
    RentalID INT PRIMARY KEY AUTO_INCREMENT,
    RenterID INT,
    TotalCost DECIMAL(10, 2),
    RentalStartDate DATE,
    RentalEndDate DATE,
    PaymentStatus ENUM('Paid', 'Pending') DEFAULT 'Pending',
    FOREIGN KEY (RenterID) REFERENCES Renter(RenterID)
);

-- RentalDetails Table
CREATE TABLE RentalDetails (
    RentalDetailID INT PRIMARY KEY AUTO_INCREMENT,
    RentalID INT,
    BikeID INT,
    RentalStartDate DATE,
    RentalEndDate DATE,
    Cost DECIMAL(10, 2),
    FOREIGN KEY (RentalID) REFERENCES RentalTransaction(RentalID),
    FOREIGN KEY (BikeID) REFERENCES Bike(BikeID)
);

-- MaintenanceStaff Table
CREATE TABLE MaintenanceStaff (
    StaffID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100),
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15)
);

-- MaintenanceRecord Table
CREATE TABLE MaintenanceRecord (
    MaintenanceID INT PRIMARY KEY AUTO_INCREMENT,
    BikeID INT,
    StaffID INT,
    MaintenanceDate DATE,
    IssueDescription TEXT,
    ResolutionDetails TEXT,
    Cost DECIMAL(10, 2),
    FOREIGN KEY (BikeID) REFERENCES Bike(BikeID),
    FOREIGN KEY (StaffID) REFERENCES MaintenanceStaff(StaffID)
);

-- Payment Table
CREATE TABLE Payment (
    PaymentID INT PRIMARY KEY AUTO_INCREMENT,
    RentalID INT,
    Amount DECIMAL(10, 2),
    PaymentDate DATE,
    Status ENUM('Paid', 'Pending') DEFAULT 'Pending',
    FOREIGN KEY (RentalID) REFERENCES RentalTransaction(RentalID)
);

-- Accountant Table
CREATE TABLE Accountant (
    AccountantID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100),
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15)
);
```

# Admin

Let's define the **Entity** and **Relationship** for the **Admin** role in the **Motorbike Rent Web Application**.

### 1. **Entity for Admin**
An **Entity** in a database represents a real-world object or concept, and in the case of the **Admin**, this entity stores information specific to administrators managing the system.

#### **Admin Entity**:
This will represent an **Admin** within the system, who manages users, rental transactions, bikes, and more.

**Admin Entity Attributes**:
- `AdminID` (Primary Key): A unique identifier for each admin.
- `Name`: The name of the admin.
- `Email`: The email address (unique).
- `Phone`: The phone number for the admin.

---

### **Admin Entity Example**:

| **Attribute**  | **Data Type**     | **Description**                                           |
|----------------|-------------------|-----------------------------------------------------------|
| `AdminID`      | INT               | Unique identifier for each admin (Primary Key).           |
| `Name`         | VARCHAR           | Name of the admin.                                        |
| `Email`        | VARCHAR (Unique)  | Unique email address of the admin.                        |
| `Phone`        | VARCHAR           | Phone number of the admin.                                |

### **Admin Entity Diagram**:
```
+------------------------+
|        Admin            |
+------------------------+
| AdminID (PK)           |
| Name                   |
| Email                  |
| Phone                  |
+------------------------+
```

---

### 2. **Relationship of Admin with Other Entities**

An **Admin** is involved in multiple actions across different entities, as they manage the entire system. Let’s define how the **Admin** relates to other entities.

#### Key Relationships Involving the Admin:

1. **Admin and User**:
   - **Relationship Type**: **One-to-Many**
   - An **Admin** manages many **Users** (renters and lenders), and can deactivate/suspend accounts or grant permissions.
   - The **Admin** does not have a direct foreign key relation in the **User** table, but the admin manages users via system permissions.

2. **Admin and Bike**:
   - **Relationship Type**: **One-to-Many**
   - An **Admin** can manage many **Bikes**. Admins can add, remove, or modify bikes in the system.

3. **Admin and Rental Transaction**:
   - **Relationship Type**: **One-to-Many**
   - An **Admin** oversees many **Rental Transactions**. Admins can view, monitor, or cancel transactions if necessary.

4. **Admin and Maintenance Record**:
   - **Relationship Type**: **One-to-Many**
   - An **Admin** manages many **Maintenance Records** to track bike issues and resolutions, and assign tasks to staff.

5. **Admin and Payment**:
   - **Relationship Type**: **One-to-Many**
   - An **Admin** manages **Payments**, ensuring all transactions are processed and can handle payment disputes.

---

### **ER Diagram for Admin and Relations**

The **Entity-Relationship Diagram (ERD)** for the **Admin** would look like this, showing relationships with other entities in the database.

```
  +------------------------+    1    +-------------------+
  |       Admin            |<-------->|       User        |
  +------------------------+          +-------------------+
  | AdminID (PK)           |          | UserID (PK)       |
  | Name                   |          | Name              |
  | Email                  |          | Email             |
  | Phone                  |          | Phone             |
  +------------------------+          +-------------------+
          | 1                                    |
          |                                      |
          |                                      |
          |                                      |
          |                                      |
  +---------------------+   1    +--------------------+      +--------------------+
  |      Bike           |<-------->| Rental Transaction |<---->|     Payment        |
  +---------------------+          +--------------------+      +--------------------+
  | BikeID (PK)         |          | RentalID (PK)      |      | PaymentID (PK)     |
  | Model               |          | RenterID (FK)      |      | Amount             |
  | Brand               |          | TotalCost          |      | Date               |
  | Year                |          | StartDate          |      | Status             |
  | ConditionStatus     |          | EndDate            |      +--------------------+
  +---------------------+          +--------------------+
          | 1                                    |
          |                                      |
          |                                      |
  +---------------------+  1    +-----------------------+
  | Maintenance Record  |<------>| Maintenance Staff      |
  +---------------------+        +-----------------------+
  | RecordID (PK)       |        | StaffID (PK)          |
  | BikeID (FK)         |        | Name                  |
  | IssueDescription    |        | Phone                 |
  | ResolutionDetails   |        | Email                 |
  | MaintenanceDate     |        +-----------------------+
  +---------------------+
```

---

### **Explanation of Relationships**:

1. **Admin ↔ User**:
   - **One-to-Many**: One **Admin** can manage many **Users** (renters, lenders).
   - **Business Logic**: Admins can create, deactivate, or suspend users.

2. **Admin ↔ Bike**:
   - **One-to-Many**: One **Admin** can manage multiple **Bikes**.
   - **Business Logic**: Admins add or remove bikes from the system.

3. **Admin ↔ Rental Transaction**:
   - **One-to-Many**: One **Admin** can oversee multiple **Rental Transactions**.
   - **Business Logic**: Admins monitor transactions, approve, or cancel rentals.

4. **Admin ↔ Maintenance Record**:
   - **One-to-Many**: One **Admin** can oversee multiple **Maintenance Records**.
   - **Business Logic**: Admin assigns or monitors bike maintenance tasks.

5. **Admin ↔ Payment**:
   - **One-to-Many**: One **Admin** manages multiple **Payments**.
   - **Business Logic**: Admins monitor payment statuses, process refunds, or resolve disputes.

---

### Summary:

- The **Admin** entity stores details about the admin, including their unique identifier, name, email, and phone number.
- The **Admin** has multiple **One-to-Many** relationships with other entities, including **User**, **Bike**, **Rental Transaction**, **Maintenance Record**, and **Payment**.
- Admins have broad control over the system, which is represented through these relationships in the database.

Would you like further details on any of these relationships or help with implementation?
