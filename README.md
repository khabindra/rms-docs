# Room and Table Management Documentation

## Current System Overview
The restaurant management system currently supports basic table management integrated with ordering, billing, and inventory functions. This document outlines current table-related models, business rules, and gaps required for a full Room/Section feature.

---

# Core Models and Relationships

## 1. Restaurant Model  
**File:** `restaurants/models.py`  
**Purpose:** Top-level container representing a restaurant location.

### Key Fields
- **name** – restaurant name  
- **owner** – ForeignKey to User  
- **is_active** – status of restaurant  

### Importance
All dependent models (tables, orders, menus, bills) link back to a Restaurant.

---

## 2. Table Model  
**File:** `orders/models.py`  
**Purpose:** Represents physical dining tables.

### Key Fields
- **table_number** – unique per restaurant  
- **status** – `Available / Occupied / Reserved / Billing`  
- **capacity** – optional seat count  
- **item_name** – table label/description  

### Relationships
- **restaurant** – ForeignKey → Restaurant

### Constraints
- **Unique (restaurant, table_number)** ensures no duplicate tables in one restaurant.

---

## 3. Bill Model  
**File:** `orders/models.py`  
**Purpose:** Represents a bill for one table session.

### Key Fields
- **table** – ForeignKey → Table (nullable for takeout/delivery)  
- **status** – `Unpaid / Paid / Cancelled / Credit`  
- **total_amount** – auto-calculated  

### Functionality
- Recalculates totals based on orders  
- Handles payments, discounts, and delivery charges  

---

## 4. Order Model  
**File:** `orders/models.py`  
**Purpose:** Represents individual orders placed at a table.

### Key Fields
- **bill** – ForeignKey → Bill  
- **table** – ForeignKey → Table  
- **status** – `Pending / Preparing / Ready / Served / Cancelled / Billed`  

### Notes
A bill may contain multiple orders for the same table session.

---

# Business Rules and Constraints

## Table Management Rules

### Table Status Flow
Available → Occupied (first order placed)
Occupied → Billing (bill requested)
Billing → Available (bill paid or credited)
Available → Reserved (reservation made)
Reserved → Available/Occupied (reservation time arrives)


### One Active Bill Per Table
A table can have multiple bills historically, but only **one active (unpaid)** bill should exist at a time.

### Automatic Status Updates
- When a bill is **paid** → table becomes **Available**
- When a bill is **credited** → table becomes **Available**

---

## Order Processing Rules

- Inventory is deducted when **OrderItem.status = Served**
- Cancelling a served item restores inventory
- Any modification to order items triggers bill recalculation

---

# Current Limitations for Room/Seating Features

### Missing Features
- No reservation system  
- No room/section model  
- No table grouping  
- No waitlist system  
- No merging/splitting tables  
- No time tracking (occupied since)  
- No customer linked to bills/tables  
- No table types (booth, bar, outdoor)  
- No features (wheelchair accessible, near window, etc.)

---


# Recommended Enhancements for Room Feature

## 1. Add Room/Section Model
```python
class Room(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    table_layout = models.JSONField(blank=True, null=True)  # optional layout data
```


## 2. Enhance Table Model

Add the following fields:

```python
room = models.ForeignKey(Room, on_delete=models.SET_NULL, null=True, blank=True)
table_type = models.CharField(
    max_length=20,
    choices=[
        ("booth", "Booth"),
        ("bar", "Bar"),
        ("standard", "Standard")
    ]
)
min_party_size = models.IntegerField(default=1)
max_party_size = models.IntegerField(default=6)
features = models.ManyToManyField("TableFeature", blank=True)
```

## 3. Add Reservation Model
```python
class Reservation(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    table = models.ForeignKey(Table, on_delete=models.CASCADE)
    customer_name = models.CharField(max_length=255)
    customer_phone = models.CharField(max_length=20)
    reservation_time = models.DateTimeField()
    party_size = models.IntegerField()
    status = models.CharField(max_length=20, choices=[
        ("confirmed", "Confirmed"),
        ("cancelled", "Cancelled"),
        ("completed", "Completed")
    ])
    notes = models.TextField(blank=True)
```

## 4. Add Waitlist Model
```python 
class WaitlistEntry(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    customer_name = models.CharField(max_length=255)
    party_size = models.IntegerField()
    estimated_wait_time = models.IntegerField()  # in minutes
    status = models.CharField(max_length=20, choices=[
        ("waiting", "Waiting"),
        ("seated", "Seated"),
        ("cancelled", "Cancelled")
    ])

```
# Next Steps for Implementation
## Phase 1

Create Room model

Add room FK to Table

## Phase 2

Implement Reservation system

## Phase 3

Add Customer profiles

## Phase 4

Implement Waitlist management

## Phase 5

Floor plan visualization (optional, frontend)

# Conclusion

The current RMS structure is strong with clear relationships:

Restaurant → Table → Bill → Order

However, it lacks room/section grouping and modern restaurant seating features.
This document provides a full analysis and roadmap for implementing advanced room and seating management.


---

Let me know if you want:

✅ A version with diagrams  
✅ A shorter summary version  
✅ A version formatted for Confluence, Notion, or GitHub documentation
