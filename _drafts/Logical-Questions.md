---

## published: false

## Coding

### Easy

* https://leetcode.com/problems/two-sum/
* https://leetcode.com/problems/valid-parentheses/description/

  * https://leetcode.com/problems/longest-common-prefix/description/

### Medium

* https://leetcode.com/problems/longest-substring-without-repeating-characters/
* https://leetcode.com/problems/merge-intervals

### Hard

* https://leetcode.com/problems/median-of-two-sorted-arrays/
* https://leetcode.com/problems/largest-rectangle-in-histogram

## Analysis/Design

* https://www.geeksforgeeks.org/aptitude/puzzle-6-monty-hall-problem/
* https://www.geeksforgeeks.org/aptitude/puzzle-2-find-ages-of-daughters/



* https://puzzles.nigelcoldwell.co.uk/eightyeight.htm





Here’s a **clean, organized Q\&A format** with **problem description**, **solution**, **key insight**, and **answer** — perfect for interviews or assessments.

---

### **1. The Light Switches**



**Question**:


You're outside a room with **3 switches**. One of them controls a **light bulb inside the room**, and the other two do nothing. You can toggle the switches however you want, but you can **enter the room only once**. How do you determine which switch controls the bulb?



**Answer**:

* Turn on **Switch A** and leave it on for 1 minute.
* Then turn it **off**, and immediately turn on **Switch B**.
* Enter the room:

  * If the bulb is **on**, it's **Switch B**.
  * If it's **off but warm**, it's **Switch A**.
  * If it's **off and cold**, it's **Switch C**.

**Key Insight**: Use **heat** to track state history, not just current status.

---

### **2. 12 Balls Puzzle**

**Question**:
You have **12 identical-looking balls**, but one is either **heavier or lighter**. You don’t know which. You may use a **balance scale 3 times**. How do you find the odd ball and whether it’s heavier or lighter?

**Answer**:

* Divide into 3 groups of 4 balls.
* Weigh 4 vs 4:

  * If equal → odd ball is in the remaining 4.
  * If not → odd is in heavier or lighter side (but you don't know which yet).

* Continue breaking down with conditional steps (3 vs 3, then 1 vs 1).

**Key Insight**: Must account for **both heavier and lighter** possibilities with every decision.

---

### **3. Bridge Crossing Problem**

**Question**:
Four people need to cross a bridge at night. They have one flashlight and the bridge can hold only **two at a time**. They take 1, 2, 5, and 10 minutes respectively to cross. When two cross, they go at the **slower** person’s pace. What's the **fastest total time** all can cross?

**Answer**:

1. 1 \& 2 cross → 2 min
2. 1 returns → 1 min
3. 5 \& 10 cross → 10 min
4. 2 returns → 2 min
5. 1 \& 2 cross again → 2 min
   **Total**: **17 minutes**

**Key Insight**: **Minimize slow crossings and fast returns**.

---

### **4. Hourglass Problem**

**Question**:
You have two hourglasses — one measures **7 minutes**, the other **11 minutes**. How can you measure **exactly 15 minutes** using them?

**Answer**:

* Start both hourglasses.
* When **7-min** runs out, flip it (7 passed).
* When **11-min** runs out, flip it (11 passed).
* When **7-min** runs out again (after second flip), total time = **15 minutes**.

**Key Insight**: Combine **time differences** through creative flipping.

---

### **5. Poison Bottle Problem**

**Question**:
You have **1000 bottles** of soda. One contains **poison** that will kill a test mouse in **24 hours**. You have **10 mice**. How can you determine the poisoned bottle in **1 day**?

**Answer**:

* Label bottles from 1 to 1000.
* Convert numbers to **binary**.
* Each mouse drinks from bottles where its bit is set (e.g., mouse 1 drinks from bottles with bit 1 = 1).
* Observe which mice die. The pattern of deaths gives the binary number of the poisoned bottle.

**Key Insight**: Use **binary representation and parallel testing**.

---

### **6. Ants on a Stick**

**Question**:
100 ants are on a **1-meter stick**, walking at the same speed in either direction. When two ants meet, they **bounce off** and reverse. When an ant reaches the end, it falls off. What's the **maximum time** before all ants are off the stick?

**Answer**:

* Collision doesn't matter — it’s like they pass through each other.
* The last ant to fall will be the one who had the **longest walk to the edge**.

**Key Insight**: **Ignore collisions** — max time = time to walk 1 meter.

---

### **7. Counterfeit Coin (8 Coins)**

**Question**:
You have **8 coins**. One is a counterfeit and is either **heavier or lighter**. Using a **balance scale only 2 times**, how do you find the fake one?

**Answer**:

* Weigh 3 vs 3.

  * If equal → odd coin is in remaining 2 → weigh 1 vs 1.
  * If unequal → choose 2 from the heavier/lighter group and weigh again to identify the fake and whether it’s heavy or light.

**Key Insight**: Plan weighings to **detect both identity and weight difference** in minimum steps.

