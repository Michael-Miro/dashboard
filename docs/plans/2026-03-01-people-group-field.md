# People Group Field Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a "קבוצה" (group) field with values A/B/C to each person, with a dropdown filter in the actions bar.

**Architecture:** Single-file vanilla JS dashboard (index.html). Add `group` to the person data model, the add/edit modal, the table display, and the filter logic in `filterPeople()`. No new files needed.

**Tech Stack:** Vanilla JS, HTML, CSS in index.html. Firebase Realtime Database + localStorage for persistence.

---

### Task 1: Add group field to person modal form

**Files:**
- Modify: `index.html:897-942` (person-modal form)

**Step 1: Add group select field to the person form**

In the person modal form, after the status field (around line 919), add:

```html
<div class="form-group">
    <label>קבוצה</label>
    <select id="person-group">
        <option value="">ללא קבוצה</option>
        <option value="A">קבוצה A</option>
        <option value="B">קבוצה B</option>
        <option value="C">קבוצה C</option>
    </select>
</div>
```

**Step 2: Verify visually**
Open index.html in browser, click "+ הוסף נהג", confirm "קבוצה" dropdown appears with A/B/C options.

---

### Task 2: Save group in savePerson() and load in editPerson()

**Files:**
- Modify: `index.html:1536-1546` (personData object in savePerson)
- Modify: `index.html:1596-1611` (editPerson function)

**Step 1: Add group to personData object in savePerson()**

Inside `savePerson()`, add `group` to the `personData` object:

```js
const personData = {
    id: personId || Date.now().toString(),
    name: document.getElementById('person-name').value,
    idNumber: document.getElementById('person-id-number').value,
    phone: document.getElementById('person-phone').value,
    status: status,
    group: document.getElementById('person-group').value || '',   // ADD THIS
    assignedCarId: newAssignedCarId,
    unit: status === 'working' ? document.getElementById('person-unit').value || '' : '',
    reference: status === 'working' ? document.getElementById('person-reference').value || '' : '',
    referenceEnd: status === 'working' ? document.getElementById('person-reference-end').value || '' : ''
};
```

**Step 2: Load group in editPerson()**

Inside `editPerson()`, after setting `person-status`, add:

```js
document.getElementById('person-group').value = person.group || '';
```

**Step 3: Verify**
Add a new person with group "B", save, reopen for edit — confirm "B" is pre-selected.

---

### Task 3: Add group column to the people table

**Files:**
- Modify: `index.html:819-835` (table header)
- Modify: `index.html:1665-1687` (renderPeople tbody)
- Modify: `index.html:1799-1821` (filterPeople tbody)

**Step 1: Add column header**

In the `<thead>` of `#people-table`, after `<th>שם מלא</th>` add:

```html
<th>קבוצה</th>
```

**Step 2: Add group cell in renderPeople()**

In the `tbody.innerHTML = people.map(person => ...)` template, after the name `<td>`, add:

```html
<td data-label="קבוצה">${person.group || '-'}</td>
```

Also update the `colspan` on the empty-state row from `9` to `10`.

**Step 3: Add group cell in filterPeople()**

Same change in the `filteredPeople.map(person => ...)` template — add the group `<td>` after name, and update `colspan` from `9` to `10`.

**Step 4: Verify**
Reload page — "קבוצה" column appears in the table. Existing people show "-".

---

### Task 4: Add group dropdown filter to the actions bar

**Files:**
- Modify: `index.html:810-817` (people actions-bar)

**Step 1: Add group filter select to the actions bar**

Replace the actions-bar div with:

```html
<div class="actions-bar">
    <button class="add-button" onclick="openModal('person')">+ הוסף נהג</button>
    <select id="filter-people-group" onchange="filterPeople()" style="padding: 8px 12px; border: 1px solid #ccc; border-radius: 6px; font-size: 14px;">
        <option value="">כל הקבוצות</option>
        <option value="A">קבוצה A</option>
        <option value="B">קבוצה B</option>
        <option value="C">קבוצה C</option>
    </select>
    <div class="search-container">
        <input type="text" class="search-box" placeholder="חיפוש נהג..." id="search-people"
            oninput="filterTable('people')">
        <button class="exit-search-btn" onclick="exitSearchMode()">סגור</button>
    </div>
</div>
```

**Step 2: Verify visually**
Reload — "כל הקבוצות" dropdown appears in the actions bar between the add button and search box.

---

### Task 5: Wire group filter into filterPeople()

**Files:**
- Modify: `index.html:1758-1822` (filterPeople function)

**Step 1: Read the group filter and apply it**

At the top of `filterPeople()`, add:

```js
const groupFilter = document.getElementById('filter-people-group').value;
```

In the `filteredPeople` filter logic, add a group condition:

```js
const filteredPeople = people.filter(person => {
    const personStatus = person.status || 'on-base';
    const matchesStatus = !statusFilter || personStatus === statusFilter;
    const matchesGroup = !groupFilter || (person.group || '') === groupFilter;
    const searchText = `${person.name} ${person.idNumber} ${person.phone} ${statusNames[personStatus]} ${person.unit || ''} ${person.reference || ''} ${person.group || ''}`.toLowerCase();
    const matchesSearch = !searchValue || searchText.includes(searchValue);
    return matchesStatus && matchesGroup && matchesSearch;
});
```

**Step 2: Verify**
Add 2 people with group A and 1 with group B. Select "קבוצה A" from dropdown — only 2 appear. Select "כל הקבוצות" — all 3 appear. Type "A" in search — only group A people appear.

---

### Task 6: Commit

```bash
git add index.html docs/plans/2026-03-01-people-group-field.md
git commit -m "Add group field (A/B/C) to people with dropdown filter"
```
