# 1. Domain

The domain is **Motorbike Rental**, which includes:

1. Stakeholders such as renters, lenders, admins, maintenance staff, and accountants.
2. Processes like bike rentals, maintenance, financial tracking, and user management.
3. Business rules and constraints specific to renting and maintaining motorbikes.



## **Database Design**

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

### E-R Diagram:
Would you like me to generate a detailed E-R Diagram for this database design?
