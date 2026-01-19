# Smart-City-Parking-Management-System
package smartcity;

import java.util.*;

// ======================= Vehicle.java =======================
abstract class Vehicle {
    protected String vehicleNo;
    protected String type;

    public Vehicle(String vehicleNo, String type) {
        this.vehicleNo = vehicleNo;
        this.type = type;
    }

    public String getVehicleNo() { return vehicleNo; }
    public String getType() { return type; }

    public abstract double calculateFare(int hours);
}

// ======================= Car.java =======================
class Car extends Vehicle {
    public Car(String vehicleNo) { super(vehicleNo, "Car"); }
    @Override
    public double calculateFare(int hours) { return hours * 50.0; }
}

// ======================= Bike.java =======================
class Bike extends Vehicle {
    public Bike(String vehicleNo) { super(vehicleNo, "Bike"); }
    @Override
    public double calculateFare(int hours) { return hours * 20.0; }
}

// ======================= ParkingSlot.java =======================
class ParkingSlot {
    private int slotId;
    private boolean occupied;
    private Vehicle vehicle;

    public ParkingSlot(int slotId) {
        this.slotId = slotId;
        this.occupied = false;
    }

    public int getSlotId() { return slotId; }
    public boolean isAvailable() { return !occupied; }

    public void occupy(Vehicle v) {
        this.vehicle = v;
        this.occupied = true;
    }

    public void release() {
        this.vehicle = null;
        this.occupied = false;
    }

    public Vehicle getVehicle() { return vehicle; }
}

// ======================= SlotNotAvailableException.java =======================
class SlotNotAvailableException extends Exception {
    public SlotNotAvailableException(String message) { super(message); }
}

// ======================= Payable.java =======================
interface Payable {
    void makePayment(String vehicleNo, double amount);
}

// ======================= ParkingManager.java =======================
class ParkingManager implements Payable {
    private List<ParkingSlot> slots;

    public ParkingManager(int totalSlots) {
        slots = new ArrayList<>();
        for (int i = 1; i <= totalSlots; i++) {
            slots.add(new ParkingSlot(i));
        }
    }

    public synchronized void parkVehicle(Vehicle v) throws SlotNotAvailableException {
        for (ParkingSlot slot : slots) {
            if (slot.isAvailable()) {
                slot.occupy(v);
                System.out.println(v.getType() + " " + v.getVehicleNo() + " parked in slot " + slot.getSlotId());
                return;
            }
        }
        throw new SlotNotAvailableException("No available parking slots!");
    }

    public synchronized void releaseVehicle(String vehicleNo, int hours) {
        for (ParkingSlot slot : slots) {
            Vehicle v = slot.getVehicle();
            if (v != null && v.getVehicleNo().equals(vehicleNo)) {
                slot.release();
                double fare = v.calculateFare(hours);
                makePayment(vehicleNo, fare);
                System.out.println("Vehicle " + vehicleNo + " released from slot " + slot.getSlotId());
                return;
            }
        }
        System.out.println("Vehicle not found in parking lot.");
    }

    public void displayStatus() {
        System.out.println("\n--- Parking Slot Status ---");
        for (ParkingSlot slot : slots) {
            if (slot.isAvailable())
                System.out.println("Slot " + slot.getSlotId() + ": Available");
            else
                System.out.println("Slot " + slot.getSlotId() + ": Occupied by " + slot.getVehicle().getVehicleNo());
        }
        System.out.println("----------------------------\n");
    }

    @Override
    public void makePayment(String vehicleNo, double amount) {
        System.out.println("Payment received for Vehicle " + vehicleNo + ": ₹" + amount);
    }
}

// ======================= MainApp.java =======================
public class MainApp {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        ParkingManager manager = new ParkingManager(5);

        System.out.println("🚗 Welcome to SmartCity Parking System 🚗");

        while (true) {
            System.out.println("\n1. Park Vehicle");
            System.out.println("2. Release Vehicle");
            System.out.println("3. View Parking Slots");
            System.out.println("4. Exit");
            System.out.print("Enter your choice: ");
            int ch = sc.nextInt();

            try {
                switch (ch) {
                    case 1:
                        System.out.print("Enter vehicle type (Car/Bike): ");
                        String type = sc.next();
                        System.out.print("Enter vehicle number: ");
                        String no = sc.next();
                        Vehicle v = type.equalsIgnoreCase("car") ? new Car(no) : new Bike(no);
                        manager.parkVehicle(v);
                        break;
                    case 2:
                        System.out.print("Enter vehicle number: ");
                        String vno = sc.next();
                        System.out.print("Enter hours parked: ");
                        int hrs = sc.nextInt();
                        manager.releaseVehicle(vno, hrs);
                        break;
                    case 3:
                        manager.displayStatus();
                        break;
                    case 4:
                        System.out.println("Thank you for using SmartCity Parking System!");
                        sc.close();
                        return;
                    default:
                        System.out.println("Invalid choice. Try again.");
                }
            } catch (SlotNotAvailableException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
