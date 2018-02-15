# General flow of rules

## Download from EmergingThreats to central

* every N minutes run rule.rulesRoutine

---

* Download tar ( url and path from env )
* Extract ( path from env )
* Read dir for filename syntax: *.rules
* Loop files:
* * Save filename as ruleset name

---

## Import to database

loop each file, line by line
* ignore comments

assign vars for:
- enabled
- sid
- revision
- classtype
- message
- severity

---

if new rule:
* if auto for this ruleset = false, set enabled false 
* * rule wont pushed out automatically
* create rule
* if ruleset has default tags, add
* if auto = false
* * create draft
* return

---

if revision has not changed && enabled has not changed
* return

---

if auto = true
* update rule
* return

---

create draft


## Draft module

if object has > 5 keys, from edit modal break down to smaller bits: 
{ ID: , KEY:  }

---

loop changes:

if new rule
* check if duplicate SID
* create draft
* return

---

get original rule

---

check if draft exists

---

if change == tags
* check if new
* if change == original, remove from draft

---

if no draft and change same as original
* return

---

if has draft and change same as draft
* return

---

if no draft, create

---

if change == tags && draft no tags
* update draft changed tags

---

if change == tags && draft has tags
* update draft changed tags

---

if change != tags
* update draft field

---

reload draft var

---

check fields that have changed compared to original
* save changed field names to draft

---

if no changed fields
* delete draft


## Draft: Publish 

if new rule
* create rule and set revision 1

---

if draft has changed fields ( new rule does not )
* loop changed fields to var
* * var.revision + 1
* update rule

---

if draft has changed tags
* loop changed tags
* check if tag still exists ( review can take time )
* * if tag added == true, and tag not found
* * * add the tag
* * if tag added == false, and tag found
* * * remove the tag

---

remove draft


## Detector - Import from central

* every N minutes run rule.checkRoutine
* get new rules
* run import based on the same mechanics as on central side 
* * run line by line check 
* * draft
* * publish