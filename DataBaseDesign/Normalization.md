# Normalization
- is the process or organizing columns and tables in a relational database to minimize data redundancy and improve data integrity.

The primary goals of normalization are:

1. **Eliminate Redundant Data:** Storing the same data multiple times wastes space and can lead to inconsistencies.
2. **Ensure Data Dependencies Make Sense:** Make sure that data is logically stored, and that related data is grouped together.
3. **Reduce Data Anomalies:** Prevent issues that can arise during data insertion, update, and deletion.


### Why Normalize? (Benefits)

- **Reduced Data Redundancy:** Less storage space is required, and there's less chance of inconsistent data.
- **Improved Data Integrity:** Ensures that data is accurate and consistent across the database.
- **Easier Maintenance:** Changes to data only need to be made in one place.
- **Better Query Performance (sometimes):** Smaller, more focused tables can sometimes lead to faster query execution, though complex joins can add overhead.
- **Elimination of Anomalies:**
    - **Insertion Anomaly:** Cannot add a new record without also adding data for another attribute. (e.g., cannot add a new course without assigning a student to it).
    - **Update Anomaly:** Changing one piece of data requires changing multiple records, leading to potential inconsistencies if not all records are updated. (e.g., changing a customer's address requires updating every order they've ever placed).
    - **Deletion Anomaly:** Deleting a record unintentionally deletes other related data. (e.g., deleting the last student from a course might also delete the course information itself).

- The Normalization going through steps:
	- 1st Normal Form.
	- 2nd Normal Form.
	- 3rd Normal Form.

- Before stating discussing the steps we have to know Functional Dependence:

## Functional Dependence
- **Dependence**: is a relation between two columns.
		<span style="color:rgb(255, 0, 0)">A--->B</span> : this mean data in Column <span style="color:rgb(255, 0, 0)">B</span> depend on the data in column <span style="color:rgb(255, 0, 0)">A</span> 

#### When we say that the table is normalized?
- When each all the columns inside the table depend only on the Primary Key.

There is three types of Dependence:
### Full Functional Dependence
- This type of Dependence appears in the table that has a composite <span style="color:rgb(255, 0, 0)">PK</span>, and all the columns in this table depend on that composite<span style="color:rgb(255, 0, 0)"> PK </span>only.
- This type could also appear in the table that has a simple PK, but with the same Condition.

### Partial Functional Dependence
- This type of Dependence appears in the table that has a composite <span style="color:rgb(255, 0, 0)">PK</span>, and there is a column in this table depend on one of the columns that consist the PK.



### Transitive Functional Dependence
- This type of Dependence appears in the table that has a column which is non Key depends on another column that is also non Key

---

## 1st Normal Form
- **Dealing with multivalued column**
- **Repeating Groups**

**Multivalued** ---> Cell that has more than one value 
				Or two columns(numbered) that have same Data like:

| Phone                       |
| --------------------------- |
| 01009395138,<br>01555195726 |

| Phone1      | Phone2      |
| ----------- | ----------- |
| 01009395138 | 01555195726 |

**Repeating Groups**: columns that repeated together and make 
				the other columns repeated.

 

| <span style="color:rgb(255, 0, 0)">ID</span> | Name  | City  | Zip | <span style="color:rgb(255, 0, 0)">CID</span> | CName | Hours | Grade | Teacher |           StPhone           |
| :------------------------------------------: | :---: | :---: | :-: | :-------------------------------------------: | :---: | :---: | :---: | :-----: | :-------------------------: |
|                      1                       | Ahme  | Cairo | 001 |                      20                       |  OS   |  70   |  90   | Hossam  | 01012111419,<br>01552383924 |
|                      1                       | Ahme  | Cairo | 001 |                      30                       |  DS   |  90   |  80   |   Ali   | 01215768419,<br>01554497924 |
|                      2                       | Omar  | Aswan | 007 |                      30                       |  DS   |  90   |  100  |   Ali   | 01045239419,<br>01124865724 |
|                      2                       | Omar  | Aswan | 007 |                      50                       |  F-E  |  60   |  90   |  Assem  | 01012111419,<br>01552383924 |
|                      2                       | Omar  | Aswan | 007 |                      40                       |  B-E  |  105  |  84   |  Saad   | 01012111419,<br>01552383924 |
|                      3                       | Abear |  Alx  | 004 |                      20                       |  OS   |  70   |  77   | Hossam  | 01012111419,<br>01552383924 |

- As we can see in this table to apply the first normal form we have to remove the <span style="color:rgb(0, 176, 80)">multivalued attributes</span>(<span style="color:rgb(255, 0, 0)">StPhone</span>) 


| <span style="color:rgb(255, 0, 0)">ID</span> | Name  | City  | Zip | <span style="color:rgb(255, 0, 0)">CID</span> | CName | Hours | Grade | Teacher |
| :------------------------------------------: | :---: | :---: | :-: | :-------------------------------------------: | :---: | :---: | :---: | :-----: |
|                      1                       | Ahme  | Cairo | 001 |                      20                       |  OS   |  70   |  90   | Hossam  |
|                      1                       | Ahme  | Cairo | 001 |                      30                       |  DS   |  90   |  80   |   Ali   |
|                      2                       | Omar  | Aswan | 007 |                      30                       |  DS   |  90   |  100  |   Ali   |
|                      2                       | Omar  | Aswan | 007 |                      50                       |  F-E  |  60   |  90   |  Assem  |
|                      2                       | Omar  | Aswan | 007 |                      40                       |  B-E  |  105  |  84   |  Saad   |
|                      3                       | Abear |  Alx  | 004 |                      20                       |  OS   |  70   |  77   | Hossam  |

| <span style="color:rgb(255, 0, 0)">ID</span> | <span style="color:rgb(255, 0, 0)">StPhone</span> |
| :------------------------------------------: | :-----------------------------------------------: |
|                      1                       |                    01012111419                    |
|                      1                       |                    01554497924                    |
|                      2                       |                    01045239419                    |
|                      2                       |                    01552383924                    |
|                      3                       |                    01012111419                    |
|                      3                       |                    01552383924                    |
- Then we have to Deal with the Repeating Groups
(<span style="color:rgb(255, 0, 0)">ID</span>, Name, City, Zip)     (<span style="color:rgb(255, 0, 0)">CID</span>, CName, Hours, Grades, Teacher, <span style="color:rgb(255, 0, 0)">SID</span>)

| <span style="color:rgb(255, 0, 0)">ID</span> | Name  | City  | Zip |
| :------------------------------------------: | :---: | :---: | :-: |
|                      1                       | Ahme  | Cairo | 001 |
|                      1                       | Ahme  | Cairo | 001 |
|                      2                       | Omar  | Aswan | 007 |
|                      2                       | Omar  | Aswan | 007 |
|                      2                       | Omar  | Aswan | 007 |
|                      3                       | Abear |  Alx  | 004 |


| <span style="color:rgb(255, 0, 0)">CID</span> | CName | Hours | Grade | Teacher | <span style="color:rgb(255, 0, 0)">SID</span> |
| :-------------------------------------------: | :---: | :---: | :---: | :-----: | :-------------------------------------------: |
|                      20                       |  OS   |  70   |  90   | Hossam  |                       1                       |
|                      30                       |  DS   |  90   |  80   |   Ali   |                       1                       |
|                      30                       |  DS   |  90   |  100  |   Ali   |                       2                       |
|                      50                       |  F-E  |  60   |  90   |  Assem  |                       2                       |
|                      40                       |  B-E  |  105  |  84   |  Saad   |                       2                       |
|                      20                       |  OS   |  70   |  77   | Hossam  |                       3                       |


--- 
## 2nd Normal Form
**Dealing with the Partial Dependency**
- In the 2nd NF we look to the tables that contain a composite PK
==> Partial Dependency Means the column that depends on one of the columns that make 
	the PK(Depend on one column only).
	 
==> In this situation we make a new table for the column and the PK column that depend 
	on it.

- In the previous example the courses table has(CName, Hours, Teacher) that depend only on the CID column which is a part of PK.

| <span style="color:rgb(255, 0, 0)">CID</span> | CName | Hours | Teacher |
| :-------------------------------------------: | :---: | :---: | :-----: |
|                      20                       |  OS   |  70   | Hossam  |
|                      30                       |  DS   |  90   |   Ali   |
|                      30                       |  DS   |  90   |   Ali   |
|                      50                       |  F-E  |  60   |  Assem  |
|                      40                       |  B-E  |  105  |  Saad   |
|                      20                       |  OS   |  70   | Hossam  |

| <span style="color:rgb(255, 0, 0)">CID</span> | <span style="color:rgb(255, 0, 0)">SID</span> | Grade |
| :-------------------------------------------: | :-------------------------------------------: | :---: |
|                      20                       |                       1                       |  90   |
|                      30                       |                       1                       |  80   |
|                      30                       |                       2                       |  100  |
|                      50                       |                       2                       |  90   |
|                      40                       |                       2                       |  84   |
|                      20                       |                       3                       |  77   |


----
## 3rd Normal Form
**Dealing with Transitive Dependency**
==> Transitive Dependency mean normal column(non-Key) depend on other normal column(non-Key).
- If we look to our example we will find that the City column depends on the zip column and vise vires.

| <span style="color:rgb(255, 0, 0)">Zip</span> | City  |
| :-------------------------------------------: | :---: |
|                      001                      | Cairo |
|                      007                      | Aswan |
|                      004                      |  Alx  |

| <span style="color:rgb(255, 0, 0)">ID</span> | Name  | Zip |
| :------------------------------------------: | :---: | :-: |
|                      1                       | Ahme  | 001 |
|                      2                       | Omar  | 007 |
|                      3                       | Abear | 004 |

--- 
## Final Result

| <span style="color:rgb(255, 0, 0)">ID</span> | <span style="color:rgb(255, 0, 0)">StPhone</span> |
| :------------------------------------------: | :-----------------------------------------------: |
|                      1                       |                    01012111419                    |
|                      1                       |                    01554497924                    |
|                      2                       |                    01045239419                    |
|                      2                       |                    01552383924                    |
|                      3                       |                    01012111419                    |
|                      3                       |                    01552383924                    |

| <span style="color:rgb(255, 0, 0)">CID</span> | CName | Hours | Teacher |
| :-------------------------------------------: | :---: | :---: | :-----: |
|                      20                       |  OS   |  70   | Hossam  |
|                      30                       |  DS   |  90   |   Ali   |
|                      30                       |  DS   |  90   |   Ali   |
|                      50                       |  F-E  |  60   |  Assem  |
|                      40                       |  B-E  |  105  |  Saad   |
|                      20                       |  OS   |  70   | Hossam  |

| <span style="color:rgb(255, 0, 0)">CID</span> | <span style="color:rgb(255, 0, 0)">SID</span> | Grade |
| :-------------------------------------------: | :-------------------------------------------: | :---: |
|                      20                       |                       1                       |  90   |
|                      30                       |                       1                       |  80   |
|                      30                       |                       2                       |  100  |
|                      50                       |                       2                       |  90   |
|                      40                       |                       2                       |  84   |
|                      20                       |                       3                       |  77   |

| <span style="color:rgb(255, 0, 0)">Zip</span> | City  |
| :-------------------------------------------: | :---: |
|                      001                      | Cairo |
|                      007                      | Aswan |
|                      004                      |  Alx  |

| <span style="color:rgb(255, 0, 0)">ID</span> | Name  | Zip |
| :------------------------------------------: | :---: | :-: |
|                      1                       | Ahme  | 001 |
|                      2                       | Omar  | 007 |
|                      3                       | Abear | 004 |
