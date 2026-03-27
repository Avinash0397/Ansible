# ASSIGNMENT 3 

 **create remote server**
 ![Pasted image (2)](https://github.com/user-attachments/assets/66d2be3f-a677-4b63-b649-ed4cdbb90b65)
 
- **create inventory**
```bash
vi inventory.ini
```
- **create playbook**
```bash
vi playbook.yml

``` 
## Project Objectives

- Automate the setup of complete infrastructure on AWS using Ansible.
- Build and deploy the [Spring3HibernateApp](https://github.com/opstree/spring3hibernate) Java application.
- Ensure end-to-end provisioning, configuration, and deployment without manual intervention.

---

### Steps Performed

1. **Install MySQL Server**  
Install and configure MySQL database required for the Spring3HibernateApp.

2. **Install Maven**  
Install Maven to build the Java application.

3. **Clone and Build Application**  

Clone the Spring3HibernateApp repository.

Build the project and generate the `.war` file using Maven.

4. **Install Java JDK 11**  

Install Java Development Kit (JDK) version 11 on the server.

5. **Install Apache Tomcat Server**  

Install and configure Apache Tomcat (version 7.0.108).

6. **Deploy WAR File**

Copy the generated `.war` file to the Tomcat `webapps` directory:
   
   ```bash
   /opt/tomcat/apache-tomcat-7.0.108/webapps/
   ```

8. **Restart Tomcat Service**  
   Restart the Tomcat server to deploy the application.

**Run playbook**
```bash
ansible-playbook -i ass-3-inventory spring3hibernate_setup.yml 
```
![Screenshot 2025-04-24 135118](https://github.com/user-attachments/assets/193fc165-3693-45fa-9e87-14038c40c71f)



## Webview
```bash
47.129.56.19:8080
```

![Screenshot 2025-04-24 140034](https://github.com/user-attachments/assets/fae0b5f6-3433-47d5-8df1-9ba569000e1b)


```bash
47.129.56.19:8080/Spring3HibernateApp/
```

![Screenshot 2025-04-24 142119](https://github.com/user-attachments/assets/519c0579-ee76-4495-a1e9-3881b52fad3e)

