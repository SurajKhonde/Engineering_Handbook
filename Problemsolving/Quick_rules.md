# Golden Rules String, Array 

## how to make right rotate by K ?

### For RIGHT rotation by k:

  - Reverse entire array
  - Reverse first k elements
  - Reverse remaining (k → n-1)

### for Left rotate by K :

- let leftRotate = n-k 
  >[!note] here n means length of array

## What is “Sort + Sweep Line” (Generic Idea)

👉 You have:
```text

- Events happening over time (or positions)
- You want to track something continuously changing
```

### Core Formula (Memorize This)
```js
1. Convert problem → events
2. Sort events
3. Sweep from left → right
4. Maintain state
5. Track answer

```

### Step 2: Sort Events


```text
time / position
```
If same time:
- Decide priority (start vs end)
### Step 3: Sweep (Traverse)
```js
for (event of events)
```
👉 Move left → right

### Step 4: Maintain State
Keep a variable like:

```js
current += event.value
```
Examples:

- number of active intervals
- current load
- current users

### Step 5: Track Answer

Inside loop:

```js
max = Math.max(max, current)
```
or whatever problem asks

### Generic Code Template

```js

function sweepLine(events) {
  // Step 1: sort events
  events.sort((a, b) => {
    if (a.time === b.time) {
      return a.type - b.type; // handle tie
    }
    return a.time - b.time;
  });

  let current = 0;
  let result = 0;

  for (let event of events) {
    current += event.value;

    // update answer
    result = Math.max(result, current);
  }

  return result;
}
```

### 3 Types of Sweep Line You’ll See

#### 🟡 Type 1: Count Overlaps (Your problem)

- Train platforms
- Meeting rooms

👉 Track max active

#### 🟡 Type 2: Merge Intervals

Overlapping intervals

👉 Track:

start, end

#### 🟡 Type 3: Range Coverage

- Skyline problem
- Area covered

👉 Track:

- active heights / values

### 🧠 When Should Your Brain Trigger This?

#### When you see

- "overlap"
- "simultaneously"
- "minimum resources"
- "active at same time"

👉 Instantly:

- “Sort + Sweep Line”

> ⚠️ Most Important Detail (Interview Killer)

- Same Time Conflict
Example:

- arrival = 1000
- departure = 1000

👉 Decide:

- arrival first → need extra
- departure first → reuse

So sorting rule matters:

(departure before arrival)
🧩 Mental Shortcut (Super Important)

Whenever stuck:

👉 Ask:

- Can I convert this into timeline events?
- If YES → Sweep Line 💥

🚀 Final Intuition

#### Think like this:

- “Instead of solving for each object,
- "I will simulate what happens over time.”


## Train Problem 

Given arrival `arr[]` and departure `dep[]` times of trains on the same day, find the minimum number of platforms needed so that no train waits. 
A platform cannot serve two trains at the same time; if a train arrives before another departs, an extra platform is needed. 

**Note**:

- Time intervals are in the 24-hour format (HHMM) , where the first two characters represent hour (between 00 to 23 ) and the last two characters represent minutes (this will be <= 59 and >= 0).
- Leading zeros for hours less than 10 are optional (e.g., 0900 is the same as 900). this how you think dont give solution how yu apprich this Examples: Input: arr[] = [900, 940, 950, 1100, 1500, 1800], dep[] = [910, 1200, 1120, 1130, 1900, 2000] 
- Output: 3 Explanation: 
  - There are three trains during the time 9:40 to 12:00. So we need a minimum of 3 platforms. 
- Input: arr[] = [900, 1235, 1100], dep[] = [1000, 1240, 1200] 
- Output: 1 
- Explanation: All train times are mutually exclusive. So we need only one platform. Input: arr[] = [1000, 935, 1100], dep[] = [1200, 1240, 1130]
-  Output: 3 
-  Explanation: All 3 trains have to be there from 11:00 to 11:30 
-  Constraints: 1 ≤ number of trains ≤ 105 0000 ≤ arr[i] ≤ dep[i] ≤ 2359


## 🧠 Step 0: Convert to Events

Given:
arr = [900, 940, 950, 1100, 1500, 1800]
dep = [910, 1200, 1120, 1130, 1900, 2000]

👉 Convert into events:

```

900  → +1
910  → -1
940  → +1
950  → +1
1100 → +1
1120 → -1
1130 → -1
1200 → -1
1500 → +1
1800 → +1
1900 → -1
2000 → -1

```

## 🧠 Step 1: Sort by Time

- Already sorted here (important step in general).

## 🧠 Step 2: Sweep the Timeline

- We move from smallest time → largest time

Track:

- current_platforms
- max_platforms

🚀 Step-by-Step Execution

```text

⏰ Time = 900
Event: +1 (train arrives)
current = 1
max = 1
⏰ Time = 910
Event: -1 (train leaves)
current = 0
max = 1
⏰ Time = 940
Event: +1
current = 1
max = 1
⏰ Time = 950
Event: +1
current = 2
max = 2
⏰ Time = 1100
Event: +1
current = 3
max = 3 🔥
⏰ Time = 1120
Event: -1
current = 2
⏰ Time = 1130
Event: -1
current = 1
⏰ Time = 1200
Event: -1
current = 0
⏰ Time = 1500
Event: +1
current = 1
⏰ Time = 1800
Event: +1
current = 2
⏰ Time = 1900
Event: -1
current = 1
⏰ Time = 2000
Event: -1
current = 0
🎯 Final Answer
max_platforms = 3

```

## 🧠 What Just Happened (Core Insight)

You didn’t:

- Compare every train ❌
- Check overlaps manually ❌

Instead:
👉 You tracked how many trains are active at each moment

🔥 Visual Mental Model

Think like this:

```text
Time → → →

Train1: 900 ─── 910
Train2:      940 ───────────── 1200
Train3:        950 ───── 1120
Train4:            1100 ── 1130
```

👉 At ~1100:

Train2
Train3
Train4

= 3 trains → 3 platforms

### 🧠 Key Pattern Locked In

Convert → Sort → Sweep → Track max

🚀 One More Important Detail (Advanced)

What if:

arrival == departure

👉 Which comes first?

- If arrival first → need extra platform
- If departure first → reuse platform

👉 So sorting rule matters:

- Departure should come before arrival at same time

🧩 Final Takeaway

Sweep line =

👉 “Turn problem into events on a timeline
👉 Move left → right
👉 Maintain running state
👉 Track peak”

```js

function findPlatform(arr, dep) {
  let n = arr.length;

  arr.sort((a, b) => a - b);
  dep.sort((a, b) => a - b);

  let i = 0, j = 0;
  let current = 0, maxPlatforms = 0;

  while (i < n && j < n) {
    if (arr[i] <= dep[j]) {
      current++;
      i++;
      if (current > maxPlatforms) maxPlatforms = current;
    } else {
      current--;
      j++;
    }
  }

  return maxPlatforms;
}

```