<table_of_contents color="gray"/>
# \[Section 5\]: YAML Introduction
# 16. Introduction to YAML
### 🔰 서버 리스트를 담은 세 가지 다른 포맷
![]()
- yaml 파일은 데이터를 나타내는 데 사용됨. (여기서는 구성 데이터)
### 🔰 YAML
![]()
- :(colon)으로 구분.
- -(dash)는 리스트를 뜻함.
- dictionary: 프로퍼티를 뜻함.
![]()
![]()
- 칸 띄우기에 따라 배치가 달라짐.
- calories 안에 fat과 carbs가 존재함.
### 🔰 YAML - ADVANCED
![]()
- 과일 \> 과일 종류 \> 과일 정보
### 🔰 DICTIONARY / LIST / LIST OF DICTIONARY
![]()
![]()
### 🔰 YAML - NOTES
- dictionary(Unordered) / List(Ordered) / Hash(Comments)
- 두 dictionary는 banana에 대한 같은 프로퍼티를 가지고 있음. 하지만, 순서가 다름.
![]()
# 17. Introduction to Coding Exercise
- 생략
# 18. Coding Exercises - Answer Keys
- Update the food.yml file to add a Vegetable - Carrot
```json
Fruit: Apple
Drink: Water
Dessert: Cake
Vegetable: Carrot
```
- Update the food.yml file to add a list of Vegetables - Carrot, Tomato, Cucumber
```json
Fruits:
  - Apple
  - Banana
  - Orange

Vegetables:
  - Carrot
  - Tomato
  - Cucumber
```
- we have updated the food.yml file with nutrition information for Fruits. Similarly update the nutrition information for Vegetables. Use the below table for information
![]()
```json
Fruits:
  - Apple:
        Calories: 95
        Fat: 0.3
        Carbs: 25
  - Banana:
      Calories: 105
      Fat: 0.4
      Carbs: 27
  - Orange:
        Calories: 45
        Fat: 0.1
        Carbs: 11

Vegetables:
  - Carrot:
        Calories: 25
        Fat: 0.1
        Carbs: 6
  - Tomato:
      Calories: 22
```
- Jacob is 30 year old Male working as a System Engineer at a firm. Represent Jacob's information (Name, Sex, Age, Title) in YAML format. Create a dictionary named Employee and define properties under it.
```json
Employee:
  Name: Jacob
  Sex: Male
  Age: 30
  Title: Systems Engineer
```
- Update the YAML file to represent the Projects assigned to Jacob. Remember Jacob works on Multiple projects - Automation and Support. So remember to use a list.
```json
Employee:
  Name: Jacob
  Sex: Male
  Age: 30
  Title: Systems Engineer
  Projects:
    - Automation
    - Support
```
- Update the YAML file to include Jacob's pay slips. Add a new property "Payslips" and create a list of pay slip detail (Use list of dictionaries). Each payslip detail contains Month and Wage.
![]()
```json
Employee:
  Name: Jacob
  Sex: Male
  Age: 30
  Title: Systems Engineer
  Projects:
    - Automation
    - Support
  Payslips:
    - Month: June
      Wage: 4000
    - Month: July
      Wage: 4500
    - Month: August
      Wage: 4000
```
