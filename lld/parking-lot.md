# Parking Lot — Low Level Design (LLD)

## Interview Context
- Round: Low Level Design (LLD)
- Level: Mid-level Backend Engineer
- Duration: 45–60 minutes
- Expectation:
  - Clear requirement gathering
  - Correct abstractions
  - Clean class design
  - Extensible but not over-engineered

---

## 1. Problem Statement (Interviewer Style)

> **Design a Parking Lot system.**

---

## 2. First Response (Very Important)

> “Before jumping into the design, I’d like to clarify the requirements and assumptions.”

This shows structured thinking. Never skip this.

---

## 3. Clarifying Questions

### Functional Questions
1. What types of vehicles should we support?
   - Car, Bike, Truck?
2. Is the parking lot multi-floor?
3. Are parking spots dedicated per vehicle type?
4. Do we need to issue a parking ticket?
5. Is fee calculation required?

### Non-Functional Questions
6. Should we handle concurrent entry/exit?
7. What is the scale? (mall-level vs small building)
8. Is persistence required or is in-memory sufficient?

---

## 4. Assumptions (Locked)

Assume interviewer confirms:

- Vehicles: **Car, Bike, Truck**
- Parking lot has **multiple floors**
- Each floor has **dedicated spots per vehicle type**
- On entry, a **ticket is generated**
- On exit, **fee is calculated based on duration**
- No external payment gateway
- In-memory design is sufficient

State clearly:
> “I’ll proceed with these assumptions.”

---

## 5. Core Use Cases

Design should be driven by use cases, not classes.

1. Park a vehicle
2. Unpark a vehicle
3. Check available spots
4. Calculate parking fee

---

## 6. Spot Allocation Strategy

Decision (keep it simple):

> “I’ll assign the **nearest available spot** for the vehicle type.”

Other strategies (not chosen):
- Random
- Any available

---

## 7. Core Entities (Nouns → Classes)

Main entities identified:

- ParkingLot
- ParkingFloor
- ParkingSpot
- Vehicle
- Ticket

---

## 8. Class-Level Responsibilities

### Vehicle
- vehicleId
- vehicleType (CAR / BIKE / TRUCK)

Subtypes:
- Car
- Bike
- Truck

Responsibility:
- Represents a vehicle entering the system

---

### ParkingSpot
- spotId
- spotType (CAR / BIKE / TRUCK)
- isOccupied
- assignedVehicle

Responsibility:
- Determines if it can fit a given vehicle
- Tracks occupancy

---

### ParkingFloor
- floorId
- List of ParkingSpot
- Tracks available spots by vehicle type

Responsibility:
- Find an available spot for a given vehicle type

---

### ParkingLot (Coordinator)
- List of ParkingFloor
- Active tickets map

Responsibility:
- Entry flow
- Exit flow
- Delegates spot allocation to floors

---

### Ticket
- ticketId
- vehicle
- parkingSpot
- entryTime
- exitTime
- fee

Responsibility:
- Tracks a single parking session

---

## 9. Fee Calculation

Keep it simple:

> “I’ll calculate fees based on the number of hours parked, with different rates per vehicle type.”

Note:
- Fee calculation logic can be abstracted later
- Avoid over-engineering early

---

## 10. Interaction Flow

### Entry Flow
1. Vehicle arrives at entry
2. ParkingLot requests a spot from floors
3. Suitable spot is assigned
4. Ticket is generated
5. Ticket is returned to the user

---

### Exit Flow
1. User presents ticket
2. System calculates parking duration
3. Fee is calculated
4. Spot is freed
5. Ticket is closed

---

## 11. Design Patterns (Mention Lightly)

Optional mentions:
- Strategy Pattern → Fee calculation
- Factory Pattern → Vehicle creation

Do NOT over-push patterns unless asked.

---

## 12. Concurrency Consideration

Mention briefly:

> “At scale, spot assignment would need synchronization to prevent double booking.”

No need to implement unless asked.

---

## 13. Handoff to Coding (STOP POINT)

End with:

> “If this design looks good, I can now start writing the class definitions and methods.”

This is the **correct transition to coding**.

---

## 14. Common Interview Mistakes (Avoid)

- Jumping straight to code
- Over-designing with DBs, APIs, microservices
- Unclear responsibilities
- Missing entry/exit flow

---

## 15. 30-Second Summary (Memorize)

> “I clarified requirements, identified core use cases, designed entities like ParkingLot, Floor, Spot, Vehicle, and Ticket, defined clear responsibilities, and walked through entry and exit flows. The design is simple, extensible, and ready for implementation.”

---
