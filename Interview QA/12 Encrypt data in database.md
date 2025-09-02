# Overview of Data Encryption

Definition: Data encryption is the process of converting plaintext data into a coded (ciphertext) format, making it unreadable to unauthorized users. This protects sensitive information at rest and in transit, ensuring confidentiality and integrity.

## Types of Encryption

- **Symmetric Encryption**:
  - In symmetric encryption, the same key is used for both encryption and decryption. This method is fast but requires secure key management.
  - Example Algorithms: AES (Advanced Encryption Standard), DES (Data Encryption Standard).

- **Asymmetric Encryption**:
  - Asymmetric encryption uses a pair of keys - a public key for encryption and a private key for decryption. This is generally slower and more complex but improves security in key distribution.
  - Example Algorithms: RSA, ECC (Elliptic Curve Cryptography).

## Implementation Procedure in a .NET Application

To demonstrate your expertise, summarize an approach to encrypting data in a SQL database using .NET:

### Choose an Encryption Strategy:
- Determine whether you need symmetric or asymmetric encryption based on your use case. For most database encryption tasks, use symmetric encryption (AES).

- Generate an Encryption Key:
    Use a secure method to generate the encryption key. Store the key securely, ideally outside of the database, for example in Azure Key Vault or using a secure configuration service.

    ```csharp
    using System.Security.Cryptography;
    // Generate a new key
    using (Aes aes = Aes.Create())
    {
        aes.GenerateKey(); // Store this key securely
        var key = aes.Key;
    }
    ```

- Encrypt Data Before Storing:
    Use the encryption algorithm to encrypt data before storing it in the database. Here’s an example of how to encrypt a string using AES:

    ```csharp
    public static byte[] EncryptStringToBytes_Aes(string plainText, byte[] Key, byte[] IV)
    {
        using (Aes aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            ICryptoTransform encryptor = aes.CreateEncryptor(aes.Key, aes.IV);
            using (MemoryStream msEncrypt = new MemoryStream())
            {
                using (CryptoStream csEncrypt 
                   = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                {
                    using (StreamWriter swEncrypt = new StreamWriter(csEncrypt))
                    {
                        swEncrypt.Write(plainText);
                    }
                    return msEncrypt.ToArray();
                }
            }
        }
    }
    ```

- Store Encrypted Data:
    Store the resulting ciphertext in your database as a binary or varbinary data type, and make sure to also store the IV (Initialization Vector) if you’re using it.

    ```sql
    INSERT INTO SensitiveData (EncryptedColumn) VALUES (@EncryptedData);
    ```

- Decrypt Data When Needed:
    Retrieve the encrypted data and decrypt it when necessary using the same key and IV. Here’s an example decryption method:

    ```csharp
    public static string DecryptStringFromBytes_Aes(
       byte[] cipherText, byte[] Key, byte[] IV)
    {
        using (Aes aes = Aes.Create())
        {
            aes.Key = Key;
            aes.IV = IV;
            ICryptoTransform decryptor = aes.CreateDecryptor(aes.Key, aes.IV);
            using (MemoryStream msDecrypt = new MemoryStream(cipherText))
            {
                using (CryptoStream csDecrypt 
                   = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
                {
                    using (StreamReader srDecrypt = new StreamReader(csDecrypt))
                    {
                        return srDecrypt.ReadToEnd();
                    }
                }
            }
        }
    }
    ```

- Consider Additional Security Measures:
    Implement additional security practices such as data masking, access controls, and regular audits to ensure that encrypted data is handled securely.

--- 

1. **Data Masking**

    - **Definition**: Data masking is a technique used to protect sensitive data by replacing it with fictitious data that retains the same format and type but does not reveal the original information. This allows development, testing, and user training to continue without exposing actual sensitive data.

    - **Use Cases**: Commonly used in development and testing environments where real data is not necessary but realistic data scenarios are required. It is critical in environments where data must be shared without disclosing sensitive information.

    - **Types of Data Masking**:
        - **Static Data Masking**: Involves creating a copy of the database and masking sensitive fields before the copy is made available to developers or testers.
        - **Dynamic Data Masking**: Masks or obfuscates sensitive data in real-time based on user roles and permissions, ensuring that only authorized users can view the actual data.

    - **Implementation Example**: In SQL Server, you can use built-in functions like CREATE TABLE with masked columns:

    ```sql
    CREATE TABLE Employees
    (
        EmployeeID INT PRIMARY KEY,
        Name NVARCHAR(100) NOT NULL,
        Salary DECIMAL(18, 2) MASKED WITH (FUNCTION = 'default()')
    );
    ```

2. **Access Controls**

    - **Definition**: Access control refers to the policies and mechanisms used to restrict who can view or use resources in a computing environment. It is essential for protecting sensitive data by ensuring that only authorized users have access to certain functionalities and information.

    - **Types of Access Control**:
        - **Role-Based Access Control (RBAC)**: Users are assigned roles, each with specific permissions to perform operations or access data. For example, an application might define roles like Admin, User, or Viewer, each with different access rights.
        - **Attribute-Based Access Control (ABAC)**: Access is granted based on attributes of the user (role, department, etc.), resource (sensitivity level), and environmental conditions (time of access, location).
        - **Mandatory Access Control (MAC)**: Access decisions are made by a central authority based on multiple candidate security levels.

    - **Implementation Example**: In ASP.NET Core, you can implement RBAC using attribute-based authorization:

    ```csharp
    [Authorize(Roles = "Admin")]
    public IActionResult AdminDashboard()
    {
        // Code for admin dashboard
    }
    ```

3. **Regular Audits**

    - **Definition**: Regular audits of data access and usage are essential for maintaining security. Audits consist of examining and evaluating the security of a database and its data processing systems.

    - **Purpose**: The goal is to identify vulnerabilities, ensure adherence to security policies, and verify that access controls are functioning as intended. Regular audits help in identifying unauthorized access, data breaches, or potential risks in the system.

    - **Types of Audits**:
        - **Security Audits**: In-depth reviews of the security measures in place, including configurations, access logs, and user permissions.
        - **Compliance Audits**: Assessment of compliance with relevant regulations and standards such as GDPR, HIPAA, or PCI DSS, ensuring that data protection policies are enforced properly.

    - **Implementation Example**: Utilize logging features in SQL Server or application logging libraries to create an audit trail:

    ```sql
    CREATE TABLE AuditLog
    (
        AuditID INT PRIMARY KEY,
        Action NVARCHAR(255),
        UserName NVARCHAR(100),
        Timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
    );
    ```
--- 
