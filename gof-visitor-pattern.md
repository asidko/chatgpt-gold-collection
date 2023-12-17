# Visitor Pattern

Useful when you have operations that need to be performed on objects of different types.

**Problem it solves:**
The Visitor pattern is ideal for scenarios where you have a set of elements with different types and you need to perform various operations on these elements. 

It decouples the operations from the objects, allowing you to add **new operations without modifying the objects**.

## Examples

### Scenario: Healthcare System


In the healthcare example, we implement a system where `Adult` and `Child` represent different patient types. 
The system applies operations like `General Checkup` and `Vaccination` to these patients

#### Without Visitor Pattern

Each type of patient has methods directly implemented for each medical procedure.

```java
class AdultPatient implements Patient {
    private String name;

    AdultPatient(String name) {
        this.name = name;
    }

    @Override
    public void performGeneralCheckup() {
        System.out.println("Performing general checkup for adult patient: " + name);
        // Checkup logic for adult
    }

    @Override
    public void scheduleVaccination() {
        System.out.println("Checking vaccination history for adult patient: " + name);
        // Vaccination logic for adult
    }
}

class ChildPatient implements Patient {
    private String name;

    ChildPatient(String name) {
        this.name = name;
    }

    @Override
    public void performGeneralCheckup() {
        System.out.println("Performing general checkup for child patient: " + name);
        // Checkup logic for child
    }

    @Override
    public void scheduleVaccination() {
        System.out.println("Scheduling vaccinations for child patient: " + name);
        // Vaccination logic for child
    }
}

// Usage
public class HealthcareDemo {
    public static void main(String[] args) {
        List<Patient> patients = Arrays.asList(new AdultPatient("Alice"), new ChildPatient("Bob"));

        for (Patient patient : patients) {
            patient.performGeneralCheckup();
            patient.scheduleVaccination();
        }
    }
}
```

**Issues with this approach:**

1. **Tight Coupling:** Each `Patient` type is tightly coupled with the logic of all medical procedures. This means changes in procedure logic require modifications to each patient class.
2. **Scalability:** Adding a new medical procedure (e.g., a specialized diagnostic test) requires updating the `Patient` interface and all its implementations, which is not scalable.
3. **Single Responsibility Principle Violation:** Each patient class is responsible for the logic of multiple procedures, which goes against the Single Responsibility Principle.


#### Use Visitor Pattern

```java
interface Patient {
    void accept(MedicalVisitor visitor);
}

class AdultPatient implements Patient {
    private String name;

    AdultPatient(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    @Override
    public void accept(MedicalVisitor visitor) {
        visitor.visit(this);
    }
}

class ChildPatient implements Patient {
    private String name;

    ChildPatient(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    @Override
    public void accept(MedicalVisitor visitor) {
        visitor.visit(this);
    }
}

interface MedicalVisitor {
    void visit(AdultPatient patient);
    void visit(ChildPatient patient);
}

class GeneralCheckupVisitor implements MedicalVisitor {
    @Override
    public void visit(AdultPatient patient) {
        System.out.println("Performing general checkup for adult patient: " + patient.getName());
        // Additional logic for adult patient checkup
    }

    @Override
    public void visit(ChildPatient patient) {
        System.out.println("Performing general checkup for child patient: " + patient.getName());
        // Additional logic for child patient checkup
    }
}

class VaccinationVisitor implements MedicalVisitor {
    @Override
    public void visit(AdultPatient patient) {
        System.out.println("Checking vaccination history for adult patient: " + patient.getName());
        // Additional logic for adult patient vaccination
    }

    @Override
    public void visit(ChildPatient patient) {
        System.out.println("Scheduling vaccinations for child patient: " + patient.getName());
        // Additional logic for child patient vaccination
    }
}

// Usage
public class HealthcareDemo {
    public static void main(String[] args) {
        List<Patient> patients = Arrays.asList(new AdultPatient("Alice"), new ChildPatient("Bob"));

        GeneralCheckupVisitor checkupVisitor = new GeneralCheckupVisitor();
        VaccinationVisitor vaccinationVisitor = new VaccinationVisitor();

        for (Patient patient : patients) {
            patient.accept(checkupVisitor);
            patient.accept(vaccinationVisitor);
        }
    }
}
```

**Advantages:**

1. **Separation of Concerns:** Each medical procedure (visitor) is a separate entity, decoupling the procedure logic from the patient classes.
2. **Ease of Adding New Procedures:** To add a new procedure, you only need to create a new visitor without modifying existing patient classes.
3. **Maintainability:** Changes in the logic of a procedure only affect the visitor, not the patient classes.


### Scenario: E-commerce Order Processing


In the e-commerce example, the system handles different sales items like Product and Service. 
It applies operations like Discount and Tax Calculation to these items, enabling easy addition of new item types and financial processes without altering the existing item structures.

#### Use Visitor pattern

```java
interface OrderItem {
    double accept(OrderVisitor visitor);
}

class Book implements OrderItem {
    private double price;

    Book(double price) {
        this.price = price;
    }

    double getPrice() {
        return price;
    }

    @Override
    public double accept(OrderVisitor visitor) {
        return visitor.visit(this);
    }
}

class Electronics implements OrderItem {
    private double price;

    Electronics(double price) {
        this.price = price;
    }

    double getPrice() {
        return price;
    }

    @Override
    public double accept(OrderVisitor visitor) {
        return visitor.visit(this);
    }
}

interface OrderVisitor {
    double visit(Book book);
    double visit(Electronics electronics);
}

class DiscountVisitor implements OrderVisitor {
    @Override
    public double visit(Book book) {
        return book.getPrice() * 0.10; // 10% discount for books
    }

    @Override
    public double visit(Electronics electronics) {
        return electronics.getPrice() * 0.05; // 5% discount for electronics
    }
}

class TaxCalculationVisitor implements OrderVisitor {
    private static final double BOOK_TAX_RATE = 0.07; // 7% tax for books
    private static final double ELECTRONICS_TAX_RATE = 0.15; // 15% tax for electronics

    @Override
    public double visit(Book book) {
        return book.getPrice() * BOOK_TAX_RATE;
    }

    @Override
    public double visit(Electronics electronics) {
        return electronics.getPrice() * ELECTRONICS_TAX_RATE;
    }
}

// Usage
public class ECommerceDemo {
    public static void main(String[] args) {
        List<OrderItem> items = Arrays.asList(new Book(29.99), new Electronics(199.99));
        double totalDiscount = 0;
        double totalTax = 0;

        DiscountVisitor discountVisitor = new DiscountVisitor();
        TaxCalculationVisitor taxCalculationVisitor = new TaxCalculationVisitor();

        for (OrderItem item : items) {
            totalDiscount += item.accept(discountVisitor);
            totalTax += item.accept(taxCalculationVisitor);
        }

        System.out.println("Total discount: " + totalDiscount);
        System.out.println("Total tax: " + totalTax);
    }
}
```

**Advantages:**

1. **Decoupling**: Financial operations like discounts and taxes are separated from the product and service classes, reducing dependencies.
2. **Easy Additions**: New financial operations can be added without altering existing item classes, enhancing adaptability.
3. **Simplified Maintenance**: Changes in financial calculations are isolated to specific visitor classes, making updates easier.
4. **Scalability**: The system can easily include new sales items, supporting business growth.
5. **Uniform Processing**: Ensures consistent application of financial rules across different types of items.
