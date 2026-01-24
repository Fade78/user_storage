# Test Prompts for User Storage v2.4

These prompts test the main features of User Storage. Run them sequentially with a CSV file attached.

## Sample CSV file (sample_sales.csv)
```csv
id,product,quantity,unit_price,date
1,Keyboard,15,49.99,2025-01-10
2,Mouse,23,29.99,2025-01-10
3,Monitor 27",8,299.00,2025-01-11
4,Webcam,12,79.50,2025-01-12
5,Headset,5,149.00,2025-01-12
6,Keyboard,10,49.99,2025-01-13
7,Mouse,18,29.99,2025-01-14
8,Monitor 27",3,299.00,2025-01-15
9,USB Hub,20,34.99,2025-01-15
10,Webcam,7,79.50,2025-01-16
```

---

## Test Sequence

### 1. Statistics
> Show me my storage space statistics.

**Expected:** Calls `store_stats()`, displays Uploads/Storage/Documents sizes.

---

### 2. Import uploaded file
> Import the CSV file I just uploaded into my space.

**Expected:** Calls `store_import(import_all=True)`, confirms import.

---

### 3. List and read
> List the imported files, then show me the first lines of the CSV and count the number of lines.

**Expected:** Lists files, shows CSV preview, counts 11 lines (10 data + 1 header).

---

### 4. Search in file
> In the imported CSV file, search for all lines containing "Monitor".

**Expected:** Uses `grep` or reads context, finds 2 lines (IDs 3 and 8).

---

### 5. Copy to Storage
> Copy the CSV file to my Storage space in a folder called "data".

**Expected:** Calls `store_uploads_to_storage(src="sample_sales.csv", dest="data/sample_sales.csv")`.

---

### 6. Create file in Storage
> In Storage, create a file called report.md with the following content:
> # Sales Report
> ## Summary
> Analysis of January 2025 sales.
> ## To Do
> - Calculate total
> - Identify best-selling product

**Expected:** Calls `store_storage_write(path="report.md", content="...")`.

---

### 7. Modify file
> Modify the file report.md in Storage to replace "To Do" with "Actions".

**Expected:** Reads file, modifies content, writes back with `store_storage_write`.

---

### 8. Read file
> Read report.md.

**Expected:** Calls `store_storage_exec(cmd="cat", args=["report.md"])`.

---

### 9. Verify modification
> Did you actually replace it with "Actions"?

**Expected:** Confirms or admits the modification wasn't applied.

---

### 10. Do it
> Do it.

**Expected:** Actually performs the modification this time.

---

### 11. Create versioned document
> Create a versioned document called "project.md" with a title and introduction. Use the commit message "Initial version".

**Expected:** Calls `store_documents_write(path="project.md", content="...", message="Initial version")`.

---

### 12. Modify and show Git history
> Modify the document project.md to add an "Objectives" section at the end. Then show me the Git history.

**Expected:** Uses `store_documents_edit_*` workflow or direct write, then `git log --oneline`.

---

### 13. Show diff
> Show me the differences between the current version and the previous version of project.md.

**Expected:** Calls `store_documents_exec(cmd="git", args=["diff", "HEAD~1", "project.md"])`.

---

### 14. Safe editing with rollback
> I want to modify report.md in Storage safely (with the ability to cancel). Open it in edit mode, replace "January" with "February", then CANCEL the modifications. Verify that the original is intact.

**Expected:** 
1. `store_storage_edit_open(path="report.md")`
2. `store_storage_edit_write(...)` with modified content
3. `store_storage_edit_cancel(path="report.md")`
4. Reads file to confirm original is intact

---

### 15. Convert with pandoc
> Convert the file report.md to HTML with pandoc and show me the result.

**Expected:** Calls `store_storage_exec(cmd="pandoc", args=["report.md", "-t", "html"])`.

---

### 16. Create archive
> Create a tar.gz archive containing the entire "data" folder in Storage.

**Expected:** Calls `store_storage_exec(cmd="tar", args=["-czf", "data.tar.gz", "data"])`.

---

### 17. Final statistics and listing
> Show me the final statistics of my space and list all files in Storage and Documents.

**Expected:** Calls `store_stats()`, then `ls -R` in both zones.

---

### 18. Cleanup
> Delete the original CSV file from Uploads to free up space.

**Expected:** Calls `store_uploads_delete(path="sample_sales.csv")`.

---

## Success Criteria

| Feature | Test # | Pass Condition |
|---------|--------|----------------|
| Import workflow | 2 | `store_import()` called BEFORE any bridge function |
| File operations | 5-8 | Read/write/copy work correctly |
| Git versioning | 11-13 | Commits created, history and diff accessible |
| Safe editing | 14 | Cancel restores original file |
| Command execution | 15-16 | pandoc and tar work |
| Security | 8 | `bash` command rejected, `cat` used instead |
