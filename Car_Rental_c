#include <stdio.h>

// Structure for Vehicle Inventory & Maintenance
struct Vehicle {
    int id;
    char model[30];
    int type;            // 1 = Hatchback, 2 = Sedan, 3 = SUV, 4 = Electric
    float dailyRate;
    int status;          // 1 = Available, 0 = Rented, 2 = Maintenance
    char serviceNote[50];
};

// Structure for Customer Details
struct Customer {
    char name[30];
    char phone[15];
    char license[20];
};

// Union for Different Rental Plans
union RentalPlan {
    int hours;           // Used if planType == 1 (Hourly)
    int days;            // Used if planType == 2 (Daily)
    int weeks;           // Used if planType == 3 (Weekly)
};

// Structure for Booking, Agreement & Billing
struct Booking {
    int bookingId;
    int vehicleId;
    struct Customer cust;
    int planType;        // 1 = Hourly, 2 = Daily, 3 = Weekly
    union RentalPlan plan;
    float totalBill;
    int returned;        // 0 = Active, 1 = Returned
};

// Function Declarations
void loadDefaultVehicles();
void viewVehicles();
void addVehicle();
void bookVehicle();
void returnAndBill();
void updateMaintenance();
void viewBookings();
float calculateRent(int vType, float baseRate, int planType, union RentalPlan plan);

int main() {
    int choice;

    // Create sample vehicles if file does not exist
    loadDefaultVehicles();

    while (1) {
        printf("\n==========================================\n");
        printf("      CAR RENTAL MANAGEMENT SYSTEM        \n");
        printf("==========================================\n");
        printf("1. View Vehicle Inventory\n");
        printf("2. Add New Vehicle\n");
        printf("3. Book a Vehicle (Rental Agreement)\n");
        printf("4. Return Vehicle & Generate Bill\n");
        printf("5. View All Bookings\n");
        printf("6. Update Vehicle Maintenance Record\n");
        printf("7. Exit\n");
        printf("------------------------------------------\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        if (choice == 1) {
            viewVehicles();
        } else if (choice == 2) {
            addVehicle();
        } else if (choice == 3) {
            bookVehicle();
        } else if (choice == 4) {
            returnAndBill();
        } else if (choice == 5) {
            viewBookings();
        } else if (choice == 6) {
            updateMaintenance();
        } else if (choice == 7) {
            printf("Thank you! Exiting program.\n");
            break;
        } else {
            printf("Invalid choice! Please enter 1 to 7.\n");
        }
    }

    return 0;
}

// Function to pre-load 3 vehicles on first run
void loadDefaultVehicles() {
    FILE *fp;
    struct Vehicle v[3] = {
        {101, "Tata_Nexon_EV", 4, 1800.0, 1, "Battery_Checked_OK"},
        {102, "Vitara_Brezza", 3, 2200.0, 1, "Oil_Service_Done"},
        {103, "Honda_City",    2, 2000.0, 1, "General_Check_OK"}
    };

    fp = fopen("vehicles.dat", "rb");
    if (fp == NULL) {
        fp = fopen("vehicles.dat", "wb");
        fwrite(v, sizeof(struct Vehicle), 3, fp);
        fclose(fp);
    } else {
        fclose(fp);
    }
}

// Function to display all vehicles
void viewVehicles() {
    FILE *fp;
    struct Vehicle v;

    fp = fopen("vehicles.dat", "rb");
    if (fp == NULL) {
        printf("No vehicles found!\n");
        return;
    }

    printf("\n--------------------------------------------------------------------------\n");
    printf("ID\tModel\t\tType\t\tRate/Day\tStatus\t\tService\n");
    printf("--------------------------------------------------------------------------\n");

    while (fread(&v, sizeof(struct Vehicle), 1, fp) == 1) {
        printf("%d\t%-15s\t", v.id, v.model);

        if (v.type == 1) printf("Hatchback\t");
        else if (v.type == 2) printf("Sedan\t\t");
        else if (v.type == 3) printf("SUV\t\t");
        else printf("Electric\t");

        printf("%.2f\t\t", v.dailyRate);

        if (v.status == 1) printf("Available\t");
        else if (v.status == 0) printf("Rented\t\t");
        else printf("Maintenance\t");

        printf("%s\n", v.serviceNote);
    }
    printf("--------------------------------------------------------------------------\n");
    fclose(fp);
}

// Function to add a new vehicle to inventory file
void addVehicle() {
    FILE *fp;
    struct Vehicle v;

    fp = fopen("vehicles.dat", "ab");
    if (fp == NULL) {
        printf("File error!\n");
        return;
    }

    printf("\nEnter Vehicle ID: ");
    scanf("%d", &v.id);
    printf("Enter Model Name (use underscore instead of space): ");
    scanf("%s", v.model);
    printf("Select Type (1-Hatchback, 2-Sedan, 3-SUV, 4-Electric): ");
    scanf("%d", &v.type);
    printf("Enter Base Daily Rate: ");
    scanf("%f", &v.dailyRate);
    printf("Enter Maintenance Note (use underscore): ");
    scanf("%s", v.serviceNote);

    v.status = 1; // Available

    fwrite(&v, sizeof(struct Vehicle), 1, fp);
    fclose(fp);
    printf("Vehicle added successfully!\n");
}

// Function to calculate charges based on duration, plan union, and vehicle type
float calculateRent(int vType, float baseRate, int planType, union RentalPlan plan) {
    float multiplier = 1.0;
    float amount = 0.0;

    // Vehicle type pricing factor
    if (vType == 3) {
        multiplier = 1.20; // SUV is 20% extra
    } else if (vType == 4) {
        multiplier = 0.90; // Electric gets 10% eco-discount (SDG 11)
    }

    // Union-based plan calculation
    if (planType == 1) {
        amount = plan.hours * (baseRate / 10.0) * multiplier;
    } else if (planType == 2) {
        amount = plan.days * baseRate * multiplier;
    } else if (planType == 3) {
        amount = plan.weeks * (baseRate * 7.0 * 0.85) * multiplier; // 15% weekly discount
    }

    return amount;
}

// Function to create a booking and rental agreement
void bookVehicle() {
    FILE *vfp, *bfp;
    struct Vehicle v;
    struct Booking b;
    int searchId, found = 0, count = 0;

    viewVehicles();

    printf("\nEnter Vehicle ID to Book: ");
    scanf("%d", &searchId);

    vfp = fopen("vehicles.dat", "rb+");
    if (vfp == NULL) {
        printf("Error opening vehicle file!\n");
        return;
    }

    while (fread(&v, sizeof(struct Vehicle), 1, vfp) == 1) {
        if (v.id == searchId) {
            found = 1;
            if (v.status != 1) {
                printf("Vehicle is not available for booking!\n");
                fclose(vfp);
                return;
            }
            break;
        }
    }

    if (found == 0) {
        printf("Vehicle ID not found!\n");
        fclose(vfp);
        return;
    }

    // Count existing bookings to generate new Booking ID
    bfp = fopen("bookings.dat", "rb");
    if (bfp != NULL) {
        while (fread(&b, sizeof(struct Booking), 1, bfp) == 1) {
            count++;
        }
        fclose(bfp);
    }

    b.bookingId = 1001 + count;
    b.vehicleId = v.id;
    b.returned = 0;

    printf("Enter Customer Name (use underscore): ");
    scanf("%s", b.cust.name);
    printf("Enter Phone Number: ");
    scanf("%s", b.cust.phone);
    printf("Enter Driving License No: ");
    scanf("%s", b.cust.license);

    printf("\nSelect Rental Plan:\n");
    printf("1. Hourly Plan\n");
    printf("2. Daily Plan\n");
    printf("3. Weekly Plan\n");
    printf("Enter Plan Choice (1-3): ");
    scanf("%d", &b.planType);

    if (b.planType == 1) {
        printf("Enter Number of Hours: ");
        scanf("%d", &b.plan.hours);
    } else if (b.planType == 2) {
        printf("Enter Number of Days: ");
        scanf("%d", &b.plan.days);
    } else if (b.planType == 3) {
        printf("Enter Number of Weeks: ");
        scanf("%d", &b.plan.weeks);
    } else {
        printf("Invalid plan!\n");
        fclose(vfp);
        return;
    }

    b.totalBill = calculateRent(v.type, v.dailyRate, b.planType, b.plan);

    // Update vehicle status to Rented (0)
    v.status = 0;
    fseek(vfp, -(long)sizeof(struct Vehicle), SEEK_CUR);
    fwrite(&v, sizeof(struct Vehicle), 1, vfp);
    fclose(vfp);

    // Save booking record
    bfp = fopen("bookings.dat", "ab");
    fwrite(&b, sizeof(struct Booking), 1, bfp);
    fclose(bfp);

    printf("\n==========================================\n");
    printf("         RENTAL AGREEMENT CONFIRMED       \n");
    printf("==========================================\n");
    printf("Booking ID   : %d\n", b.bookingId);
    printf("Customer     : %s\n", b.cust.name);
    printf("License No   : %s\n", b.cust.license);
    printf("Vehicle      : %s (ID: %d)\n", v.model, v.id);
    printf("Estimated Amt: Rs. %.2f\n", b.totalBill);
    printf("==========================================\n");
}

// Function to return vehicle and generate final bill
void returnAndBill() {
    FILE *bfp, *vfp;
    struct Booking b;
    struct Vehicle v;
    int searchBookingId, found = 0;
    float extraFee, tax, finalAmount;

    printf("\nEnter Booking ID to Return: ");
    scanf("%d", &searchBookingId);

    bfp = fopen("bookings.dat", "rb+");
    if (bfp == NULL) {
        printf("No bookings found!\n");
        return;
    }

    while (fread(&b, sizeof(struct Booking), 1, bfp) == 1) {
        if (b.bookingId == searchBookingId) {
            found = 1;
            if (b.returned == 1) {
                printf("This vehicle has already been returned!\n");
                fclose(bfp);
                return;
            }
            break;
        }
    }

    if (found == 0) {
        printf("Booking ID not found!\n");
        fclose(bfp);
        return;
    }

    printf("Enter Late / Damage Charges (0 if none): ");
    scanf("%f", &extraFee);

    tax = (b.totalBill + extraFee) * 0.18; // 18% GST
    finalAmount = b.totalBill + extraFee + tax;

    b.totalBill = finalAmount;
    b.returned = 1;

    fseek(bfp, -(long)sizeof(struct Booking), SEEK_CUR);
    fwrite(&b, sizeof(struct Booking), 1, bfp);
    fclose(bfp);

    // Make vehicle available again
    vfp = fopen("vehicles.dat", "rb+");
    while (fread(&v, sizeof(struct Vehicle), 1, vfp) == 1) {
        if (v.id == b.vehicleId) {
            v.status = 1;
            fseek(vfp, -(long)sizeof(struct Vehicle), SEEK_CUR);
            fwrite(&v, sizeof(struct Vehicle), 1, vfp);
            break;
        }
    }
    fclose(vfp);

    printf("\n==========================================\n");
    printf("              FINAL RENTAL BILL           \n");
    printf("==========================================\n");
    printf("Booking ID    : %d\n", b.bookingId);
    printf("Customer Name : %s\n", b.cust.name);
    printf("Vehicle ID    : %d\n", b.vehicleId);
    printf("Extra Charges : Rs. %.2f\n", extraFee);
    printf("GST (18%%)     : Rs. %.2f\n", tax);
    printf("Total Payable : Rs. %.2f\n", finalAmount);
    printf("==========================================\n");
}

// Function to view all booking records
void viewBookings() {
    FILE *bfp;
    struct Booking b;

    bfp = fopen("bookings.dat", "rb");
    if (bfp == NULL) {
        printf("No booking records found!\n");
        return;
    }

    printf("\n-----------------------------------------------------------------\n");
    printf("BookID\tVehID\tCustomer\tPlan\tTotal Bill\tStatus\n");
    printf("-----------------------------------------------------------------\n");

    while (fread(&b, sizeof(struct Booking), 1, bfp) == 1) {
        printf("%d\t%d\t%-12s\t", b.bookingId, b.vehicleId, b.cust.name);

        if (b.planType == 1) printf("Hourly\t");
        else if (b.planType == 2) printf("Daily\t");
        else printf("Weekly\t");

        printf("%.2f\t\t", b.totalBill);

        if (b.returned == 1) printf("Returned\n");
        else printf("Active\n");
    }
    printf("-----------------------------------------------------------------\n");
    fclose(bfp);
}

// Function to update vehicle maintenance status and logs
void updateMaintenance() {
    FILE *vfp, *mfp;
    struct Vehicle v;
    int searchId, found = 0, newStatus;
    float serviceCost;

    viewVehicles();
    printf("\nEnter Vehicle ID for Maintenance: ");
    scanf("%d", &searchId);

    vfp = fopen("vehicles.dat", "rb+");
    if (vfp == NULL) {
        printf("File error!\n");
        return;
    }

    while (fread(&v, sizeof(struct Vehicle), 1, vfp) == 1) {
        if (v.id == searchId) {
            found = 1;
            break;
        }
    }

    if (found == 0) {
        printf("Vehicle ID not found!\n");
        fclose(vfp);
        return;
    }

    printf("Enter Service Details (use underscore): ");
    scanf("%s", v.serviceNote);
    printf("Enter Service Cost: ");
    scanf("%f", &serviceCost);
    printf("Set Status (1 = Ready/Available, 2 = Under Maintenance): ");
    scanf("%d", &newStatus);

    v.status = newStatus;
    fseek(vfp, -(long)sizeof(struct Vehicle), SEEK_CUR);
    fwrite(&v, sizeof(struct Vehicle), 1, vfp);
    fclose(vfp);

    // Save maintenance record to text file
    mfp = fopen("maintenance.txt", "a");
    if (mfp != NULL) {
        fprintf(mfp, "VehicleID: %d | Model: %s | Note: %s | Cost: %.2f\n",
                v.id, v.model, v.serviceNote, serviceCost);
        fclose(mfp);
    }

    printf("Maintenance record updated and saved!\n");
}
