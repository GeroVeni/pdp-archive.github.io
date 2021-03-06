---
layout: solution
codename: ensemble
---

## Λύση

Ο μέσος όρος των στοιχείων $$Α[0], .. ., Α[Ν-1]$$ είναι 
$$\frac{A[0]+ ... + A[N-1]}{N}$$

Για τον Α) τύπου μέσο όρο, πρέπει να αφαιρέσουμε τις $$Κ$$ μικρότερες και τις $$Κ$$ μεγαλύτερες τιμές και να πάρουμε τον μέσο όρο των υπόλοιπων στοιχείων. Ο βέλτιστος τρόπος για να το κάνουμε αυτό είναι ταξινομώντας τον πίνακα και υπολογίζοντας τον μέσο όρο των $$A[K], ..., A[N-K-1]$$. 

Για το πρώτο παράδειγμα, ο ταξινομημένος πίνακας είναι:

$$5.6, 6.2, \mathbf{6.4, 7.8, 8.2, 8.4, 8.5, 9.1}, 9.2, 9.3$$

και θέλουμε τον μέσο όρο των $$N-2*K$$ bold στοιχείων (που είναι $$8.07$$).

Για τον Β) τύπου μέσο όρο, πρέπει να αντικαταστήσουμε τις $$Κ$$ μικρότερες και τις $$Κ$$ μεγαλύτερες τιμές με τις κοντινότερές τους τιμές. Για τις μικρότερες τιμές αυτή είναι η $$Α[Κ]$$ ($$=6.4$$ στο παράδειγμα) και η $$Α[Ν-Κ-2]$$ ($$= 9.1$$ στο παράδειγμα).

Ο αλγόριθμος έχει πολυπλοκότητα $$O(N \log N)$$ (λόγω της ταξινόμησης) και μνήμη $$O(N)$$.

**Σημείωση:** Η συνθήκη $$k \leq \frac{N-1}{2}$$ εγγυάται ότι αφού αφαιρέσουμε $$2K$$ στοιχεία, ο πίνακας δεν θα είναι άδειος.

{% highlight c++ %}
#include <algorithm>
#include <cstdio>

const long MAXN = 100000;

double A[MAXN];

int main() {
  FILE *fi = fopen("ensemble.in", "r");
  long N, K;
  fscanf(fi, "%ld %ld", &N, &K);
  
  for (long i = 0; i < N; ++i) {
    fscanf(fi, "%lf", &A[i]);
  }
  
  std::sort(A, A+N);
  
  double sumA = 0.0;
  double sumB = 0.0;
  
  for (long i = 0; i < N; ++i) {
    if (i < K) {
      sumB += A[K];
    } else if (i > N - K -1) {
      sumB += A[N-K-1];
    } else {
      sumA += A[i];
      sumB += A[i];
    }
  }
  fclose(fi);
  
  FILE *fo = fopen("ensemble.out", "w");
  double averageA = sumA / double(N - 2 * K);
  double averageB = sumB / double(N);
  fprintf(fo, "%.2lf %.2lf\n", averageA, averageB);
  fclose(fo);
  return 0;
}
{% endhighlight %}

## Τέλεια ακρίβεια (προαιρετικό)
Αν χρησιμοποιήσουμε floats αντί για doubles, τότε λόγω περιορισμένης ακρίβειας θα χάσουμε ένα testcase. Αν και στο συγκεκριμένο πρόβλημα είναι αρκετό να χρησιμοποιήσουμε doubles, μπορούμε να αποφύγουμε τελείως την ανακριβή αναπαράσταση. 

Αφού κάθε στοιχείο έχει ακρίβεια ενός δεκαδικού ψηφίου, το άθροισμα θα έχει πάντοτε ακρίβεια ενός δεκαδικού ψηφίου. Για την διαίρεση μας ενδιαφέρουν δύο δεκαδικά στοιχεία στο αποτέλεσμα, άρα πρέπει να υπολογίσουμε με ακρίβεια τριών και να στρογγυλοποιήσουμε αναλόγως. Συνεπώς, χρειαζόμαστε τρία δεκαδικά ψηφία. 

Το τρικ είναι να πολλαπλασιάσουμε επί $$1000$$, ώστε τα στοιχεία να είναι ακέραιοι με την ελάχιστη ακρίβεια που χρειαζόμαστε. Αθροίζουμε κανονικά και έπειτα διαιρούμε με το πλήθος των στοιχείων. Στο παράδειγμα θα πάρουμε άθροισμα $$48400$$ και μέσο όρο $$8066$$. Αποκωδικοποιώντας την απάντηση, παίρνουμε $$8.066$$. Βλέπουμε ότι το τελευταίο ψηφίο είναι μεγαλύτερο ή ίσο από $$5$$ και προσθέτουμε $$0.01$$ στο υπόλοιπο, λαμβάνοντας $$8.07$$. Δείτε τον παρακάτω κώδικα για λεπτομέρειες.

{% highlight c++ %}
#include <algorithm>
#include <cstdio>

const long MAXN = 100000;

long A[MAXN];

void printValue(FILE *fo, long val);

int main() {
  FILE *fi = fopen("ensemble.in", "r");
  long N, K;
  fscanf(fi, "%ld %ld", &N, &K);
  
  for (long i = 0; i < N; ++i) {
    double temp;
    fscanf(fi, "%lf", &temp);
    A[i] = temp * 1000;
  }
  
  std::sort(A, A+N);
  
  long sumA = 0;
  long sumB = 0;
  
  for (long i = 0; i < N; ++i) {
    if (i < K) {
      sumB += A[K];
    } else if (i > N - K -1) {
      sumB += A[N-K-1];
    } else {
      sumA += A[i];
      sumB += A[i];
    }
  }
  fclose(fi);
  
  FILE *fo = fopen("ensemble.out", "w");
  long averageA = sumA / (N - 2 * K);
  long averageB = sumB / N;
  printValue(fo, averageA);
  fprintf(fo, " ");
  printValue(fo, averageB);
  fprintf(fo, "\n");
  fclose(fo);
  return 0;
}

void printValue(FILE *fo, long val) {
  long decimal = val / 10;
  // Προσθέτουμε 0.01
  if (val % 10 >= 5) {
    decimal++;
  }
  long first_two = decimal / 100;
  long last_two = decimal % 100;
  fprintf(fo, "%ld.%02ld", first_two, last_two);
}
{% endhighlight %}
