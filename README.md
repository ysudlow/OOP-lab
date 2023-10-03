import java.text.SimpleDateFormat;
import java.util.*;

class Book {
    String title;
    String author;
    String ISBN;
    boolean isCheckedOut;
    String dueDate;

    // Constructor for Book class
    Book(String title, String author, String ISBN) {
        this.title = title;
        this.author = author;
        this.ISBN = ISBN;
        this.isCheckedOut = false;
        this.dueDate = "";
    }

    @Override
    public String toString() {
        return "Title: " + title + ", Author: " + author + ", ISBN: " + ISBN;
    }
}

class Member {
    String name;
    String memberID;
    String email;
    List<Book> checkedOutBooks;

    // Constructor for Member class
    Member(String name, String memberID, String email) {
        this.name = name;
        this.memberID = memberID;
        this.email = email;
        this.checkedOutBooks = new ArrayList<>();
    }
}

class Library {
    Map<String, Book> books;
    Map<String, Member> members;
    Member currentMember;

    // Constructor for Library class
    Library() {
        this.books = new HashMap<>();
        this.members = new HashMap<>();
    }

    // Method to add a book to the library
    void addBook(Book book) {
        books.put(book.ISBN, book);
    }

    // Method to add a member to the library
    void addMember(Member member) {
        members.put(member.memberID, member);
    }

    // Method to login a member
    Member login(String email) {
        for (Member member : members.values()) {
            if (member.email.equals(email)) {
                currentMember = member;
                return member;
            }
        }
        return null;
    }

    // Method to display popular books
    void displayPopularBooks() {
        System.out.println("Popular Books:");
        int i = 1;
        for (Book book : books.values()) {
            System.out.println(i + ". " + book);
            i++;
        }
    }

    // Method to get a book by its number
    Book getBookByNumber(int number) {
        int i = 1;
        for (Book book : books.values()) {
            if (i == number) {
                return book;
            }
            i++;
        }
        return null;
    }

// Method to checkout a book
void checkoutBook(int bookNumber, String memberID) {
    Book book = getBookByNumber(bookNumber);
    Member member = members.get(memberID);
    if (book != null && member != null && !book.isCheckedOut) {
        book.isCheckedOut = true;
        Calendar cal = Calendar.getInstance();
        cal.add(Calendar.DATE, 14);  // Due date is 14 days from now
        book.dueDate = new SimpleDateFormat("yyyy-MM-dd").format(cal.getTime());
        member.checkedOutBooks.add(book);
        System.out.println(member.name + " checked out " + book.title + ". Due date: " + book.dueDate);
        System.out.println("A reminder email to extend or return will be sent to " + member.email + " 1 week prior to the due date.");  // Updated message
    } else if (book == null) {
        System.out.println("Invalid book number. Please try again.");
    } else {
        System.out.println("Checkout failed. This book is already checked out.");
    }
}

      // Method to return a book
    void returnBook(int bookNumber, String memberID) {
        Book book = getBookByNumber(bookNumber);
        Member member = members.get(memberID);
        if (book != null && member != null && book.isCheckedOut) {
            book.isCheckedOut = false;
            book.dueDate = "";
            member.checkedOutBooks.remove(book);
            String timestamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
            System.out.println(member.name + " returned " + book.title + " at " + timestamp + ".");
        } else if (book == null) {
            System.out.println("Invalid book number. Please try again.");
        } else {
            System.out.println("Return failed. Either the book or member does not exist, or the book was not checked out.");
        }
    }

}

class LibraryApp {
    public static void main(String[] args) {
        Library library = new Library();
        Scanner scanner = new Scanner(System.in);

        // Add some initial data
        library.addBook(new Book("To Kill a Mockingbird", "Harper Lee", "978-0-06-112008-4"));
        library.addBook(new Book("1984", "George Orwell", "978-0-452-28423-4"));
        library.addBook(new Book("The Great Gatsby", "F. Scott Fitzgerald", "978-0-7432-7356-5"));
        library.addBook(new Book("The Catcher in the Rye", "J.D. Salinger", "978-0-316-76948-0"));
        library.addBook(new Book("Moby Dick", "Herman Melville", "978-0-553-21311-7"));
        library.addMember(new Member("John Doe", "1234567890", "john.doe@example.com"));

        try {
            while (true) {
                System.out.println("Welcome to the Library Checkout System!");
                System.out.println("Are you a new or existing member?");
                System.out.println("1. New Member");
                System.out.println("2. Existing Member");
                System.out.println("3. Exit");
                System.out.print("Enter your choice: ");
                int choice = scanner.nextInt();
                scanner.nextLine();  // Consume newline

                switch (choice) {
                    case 1:
                        System.out.print("Enter Member Name: ");
                        String memberName = scanner.nextLine();
                        String memberID;
                        while (true) {  // Loop until a valid Member ID is entered
                            System.out.print("Enter Member ID (Phone): ");
                            memberID = scanner.nextLine();
                            if (isValidMemberID(memberID)) {
                                break;  // Exit loop if Member ID is valid
                            }
                            System.out.println("Invalid Member ID. Please enter a 10-digit phone number.");
                        }
                        System.out.print("Enter Email: ");
                        String email = scanner.nextLine();
                        Member newMember = new Member(memberName, memberID, email);
                        library.addMember(newMember);
                        System.out.println("New member added: " + memberName + ", Member ID: " + memberID);
                        library.currentMember = newMember;
                        memberMenu(library, scanner, newMember);
                        break;
                    case 2:
                        System.out.print("Enter Email: ");
                        String loginEmail = scanner.nextLine();
                        Member member = library.login(loginEmail);
                        if (member != null) {
                            System.out.println("Login successful. Welcome, " + member.name + "!");
                            memberMenu(library, scanner, member);
                        } else {
                            System.out.println("Login failed. No member found with that email.");
                        }
                        break;
                    case 3:
                        System.out.println("Goodbye!");
                        scanner.close();
                        System.exit(0);
                        break;
                    default:
                        System.out.println("Invalid choice. Please try again.");
                        break;
                }
            }
        } catch (InputMismatchException e) {
            System.out.println("Error: Invalid input. Please enter a number.");
            scanner.nextLine();  // Clear the buffer
        } catch (Exception e) {
            System.out.println("An unexpected error occurred: " + e.getMessage());
        } finally {
            scanner.close();  // Close the scanner to prevent memory leaks
        }
    }

    // New method to validate Member IDs
    public static boolean isValidMemberID(String memberID) {
        return memberID.matches("\\d{10}");  // Returns true if memberID is exactly 10 digits
    }

    // New method to handle member-specific actions
    public static void memberMenu(Library library, Scanner scanner, Member member) {
        try {
            while (true) {
                System.out.println("Member Menu:");
                System.out.println("1. Checkout Book");
                System.out.println("2. Return Book");
                System.out.println("3. Logout");
                System.out.print("Enter your choice: ");
                int choice = scanner.nextInt();
                scanner.nextLine();  // Consume newline

                switch (choice) {
                    case 1:
                        library.displayPopularBooks();
                        System.out.print("Enter book number: ");
                        int checkoutBookNumber = scanner.nextInt();
                        scanner.nextLine();  // Consume newline
                        library.checkoutBook(checkoutBookNumber, member.memberID);
                        break;
                    case 2:
                        System.out.print("Enter book number: ");
                        int returnBookNumber = scanner.nextInt();
                        scanner.nextLine();  // Consume newline
                        library.returnBook(returnBookNumber, member.memberID);
                        break;
                    case 3:
                        System.out.println("Logged out. Thank you for reading! Goodbye, " + member.name + "!");
                        return;
                    default:
                        System.out.println("Invalid choice. Please try again.");
                        break;
                }
            }
        } catch (InputMismatchException e) {
            System.out.println("Error: Invalid input. Please enter a number.");
            scanner.nextLine();  // Clear the buffer
        } catch (Exception e) {
            System.out.println("An unexpected error occurred: " + e.getMessage());
        }
    }
}
