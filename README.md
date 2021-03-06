# Διασύνδεση με την υπηρεσία Click Away της πλατφόρμας ekatanalotis

## Εισαγωγή 

Η υπηρεσία παρέχεται δωρεάν στους ιδιοκτήτες ηλεκτρονικών καταστημάτων με σκοπό την υποστήριξη τους στην οργάνωση των ραντεβού και την αποστολή τους προς τους πελάτες τους. 

## Έκδοση κλειδιού

Για την έκδοση κλειδιού πρόσβασης στην προγραμματιστική διεπαφή ο έμπορος θα χρειαστεί να επισκεφθεί την ιστοσελίδα https://ekatanalotis.gov.gr και να επιλέξει "Εγγραφή eShop". Αφού συμπληρώσει τα στοιχεία του θα παραλάβει αυτόματα στο ηλεκτρονικό ταχυδρομείο του τα στοιχεία πρόσβασης, τα οποία είναι τα παρακάτω:
- **web_id:** ανωνυμοποιημένο αλφαριθμητικό αναγνώρισης εμπόρου 
- **api_key:** συνθηματικό πρόσβασης στην υπηρεσία 

## Πρόσβαση API

Με τα παραπάνω στοιχεία θα χρειαστεί να γίνει μια κλήση με τη μέθοδο `POST` στο endpoint:

```
https://app.ekatanalotis.gov.gr/api/mobile/v2/ed840ad545884deeb6c6b699176797ed/context/
```

προσθέτωντας τους παρακάτω headers στην κλήση:
- **loyalty-date:** Ημερομηνία που γίνεται η κλήση στη μορφή `ΥΥΥΥ-ΜΜ-DD HH:MM:ss`, θα πρέπει να είναι η ίδια με την ημερομηνία και ώρα του παρακάτω πεδίου 
- **loyalty-signature:** Η SHA256 αναπαράσταση του αλφαριθμητικού που προκύπτει από την ένωση του `api_key` και της ημερομηνίας και ώρας πχ.`e22d39139f0248b992314191e50f17572020-12-18 12:37:31`  
- **loyalty-web-id:** To  `web_id` που ήρθε στο ηλεκτρονικό ταχυδρομείο κατά την εγγραφής

Η δημιουργία των παραπάνω κλειδιών προκύπτει σε Python 

```python
def generate_headers_from_keys(web_id,api_key):
    m = hashlib.sha256()
    now = datetime.datetime.utcnow()
    m.update((api_key + str(now).split(".")[0]).encode('utf-8'))
    print('loyalty-date:' + str(now).split(".")[0])
    print('loyalty-signature:' + str(m.hexdigest()))
    print('loyalty-web-id:' + str(web_id))
```

ενώ σε Javascript έχουμε:

```javascript
    var now = new Date();
    var date_format = now.getFullYear() + '-' + now.getMonth() + '-' + now.getDate() + ' ' + now.getHours() + ':' + now.getMinutes() + ':' + now.getSeconds();
    xhr.setRequestHeader('loyalty-web-id', web_id);
    xhr.setRequestHeader('loyalty-date', date_format);
    xhr.setRequestHeader('loyalty-signature', sha256_digest(api_key) + date_format));
```

## Λειτουργίες API

Οι λειτουργίες που υποστηρίζονται είναι οι εξής:

1.Δημιουργία ραντεβού για παραλαβή Click Away

- POST: https://app.ekatanalotis.gov.gr/api/mobile/v2/ed840ad545884deeb6c6b699176797ed/context/
- HEADERS: όπως προσδιορίζονται στην προηγούμενη παράγραφο
- BODY:
```
{
    "products": 
    {
        "action": "click_away",
        "subaction": "add_appointment",
        "code": "358515",
        "extra_fields": 
        {
            "merchant":"name",
            "afm":"afm",
            "address":"address",
            "date":"2020-12-18",
            "time":"11:00",
            "comments":""
        }
    }
}
```

Όπου τα πεδία έχουν τις τιμές:
- **merchant:** Η επωνυμία του εμπόρου, η οποία είναι απαραίτητη από την Κ.Υ.Α. σε περίπτωση ελέγχου
- **afm:** Ο Α.Φ.Μ. του εμπόρου, ο οποίος είναι απαραίτητος απο την Κ.Υ.Α. σε περιπτωση ελέγχου
- **address:** Η διεύθυνση του καταστήματος παραλαβής με τη μέθοδο Click Away, η οποία είναι απαραίτητη από την Κ.Υ.Α. σε περίπτωση ελέγχου
- **code:** Ο κωδικός ραντεβού που δημιουργείται μέσα από το ekatanalotis app και χρειάζεται να τον συμπληρώσει ο καταναλωτής στη σελίδα του checkout. 
- **date:** Η ημερομηνία του ραντεβού με τη κωδικοποίηση `ΥΥΥΥ-MM-DD`
- **time:** Η ώρα του ραντεβού με την κωδικοποίηση `ΗΗ:MM` 
- **commments:** Σχόλια για την παράδοση, τα οποία είανι εμφανή στον καταναλωτή μέσα από την εφαρμογή ekatanalotis, στην ενότητα **Τα Ραντεβού Μου**


2.Ανάκτηση όλων των ραντεβού

- POST: https://app.ekatanalotis.gov.gr/api/mobile/v2/ed840ad545884deeb6c6b699176797ed/context/
- HEADERS: όπως προσδιορίζονται στην προηγούμενη παράγραφο
- BODY:
```
{
    "products":
    {
        "action":"click_away",
        "subaction":"get_appointments"
    }
}
```

2. Διαγραφή ραντεβού 

- POST: https://app.ekatanalotis.gov.gr/api/mobile/v2/ed840ad545884deeb6c6b699176797ed/context/
- HEADERS: όπως προσδιορίζονται στην προηγούμενη παράγραφο
- BODY:
```
{
    "products":
    {
        "action":"click_away",
        "subaction":"remove_appointment",
       "code":"358515"
    }
}
```

Επισημαίνουμε ότι το ραντεβού με αυτή τη μέθοδο διαγράφεταί μόνο από την πλευρά του εμπόρου.

## Υποστήριξη 

Έχουμε προσθέσει τα παρακάτω εργαλεία στη φαρέτρα του προγραμματιστή ώστε να μπορέσει να πειραματιστεί και να δοκιμάσει την πρόσβαση κατά τη διαδικασία υλοποίησης. Τα εργαλεία αυτά είναι:

1. Μια συλλογή σε Postman των κλήσεων και της μορφής που θα πρέπει να έχουν. H συλλογή Postman είναι κομμάτι του απουετηρίου. 
2. Για τον έλεγχο του σωστού υπολογισμού του hash μπορείτε να χρησιμοποιήσετε το αρχείο `validate.py`, το οποίο αποτελεί αναπόσπαστο κομμάτι του συγκεκριμένου αποθετηρίου. 

```python
python validate.py <loyalty-web-id> <loyalty-signature> <loyalty-date> <api_key>
```

Για παράδειγμα:

```$ python
python validate.py 70ee27ddec8f4c2788fe86be914009bf \ 
787861a4c577a0123e947b001233fb12fe357ba37466016e3f0e13aac80027e7 \ 
'2020-12-18 12:37:31' e22d39139f0248b992314191e50f1757
```

Για οποιαδήποτε απορία και θέμα κατά τη διάρκεια της διαδικασίας έκδοσης και διασύνδεσης παρακαλώ επικοινωνήστε με την τεχνική ομάδα στην ηλεκτρονική διεύθυνση  ekatanalotisapp at gmail com.
