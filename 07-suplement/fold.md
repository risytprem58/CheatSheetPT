# =========================================================
# UJI SIGNIFIKANSI STATISTIK: SVM TERBAIK vs KNN TERBAIK
# =========================================================

import numpy as np
from scipy import stats
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import Pipeline
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier

print("Menjalankan pengujian 5-fold CV untuk SVM dan KNN...")

# 1. Pipeline SVM Terbaik (Peringkat 1 di Tabel 4)
pipe_svm = Pipeline([
    ('preprocessing', preprocessor),
    ('model', SVC(
        kernel='poly', 
        C=10, 
        gamma='scale', 
        degree=3, 
        class_weight='balanced', 
        random_state=42
    ))
])

# 2. Pipeline KNN Terbaik (Peringkat 8 di Tabel 4)
pipe_knn = Pipeline([
    ('preprocessing', preprocessor),
    ('model', KNeighborsClassifier(
        n_neighbors=3, 
        weights='distance', 
        metric='manhattan'
    ))
])

# 3. Hitung F1-score untuk masing-masing 5 fold (scoring macro)
scores_svm = cross_val_score(pipe_svm, X, y_encoded, cv=cv, scoring='f1_macro', n_jobs=-1)
scores_knn = cross_val_score(pipe_knn, X, y_encoded, cv=cv, scoring='f1_macro', n_jobs=-1)

# 4. Uji Signifikansi Statistik
# A. Paired t-test
t_stat, p_val_ttest = stats.ttest_rel(scores_svm, scores_knn)

# B. Wilcoxon Signed-Rank Test
w_stat, p_val_wilcoxon = stats.wilcoxon(scores_svm, scores_knn)

# =========================================================
# OUTPUT HASIL
# =========================================================
print("\n" + "="*50)
print("HASIL PENGUJIAN LINTAS 5 FOLD")
print("="*50)
print(f"F1-score SVM per fold : {np.round(scores_svm, 4)}")
print(f"Rata-rata F1 SVM      : {scores_svm.mean():.4f} (± {scores_svm.std():.4f})")
print("-" * 50)
print(f"F1-score KNN per fold : {np.round(scores_knn, 4)}")
print(f"Rata-rata F1 KNN      : {scores_knn.mean():.4f} (± {scores_knn.std():.4f})")
print("="*50)
print("HASIL UJI SIGNIFIKANSI STATISTIK:")
print(f"1. Paired t-test : t = {t_stat:.4f}, p-value = {p_val_ttest:.6f}")
print(f"2. Wilcoxon Test : W = {w_stat:.4f}, p-value = {p_val_wilcoxon:.6f}")
print("="*50)

if p_val_ttest < 0.05:
    print("KESIMPULAN: Perbedaan performa SIGNIFIKAN secara statistik (p < 0.05).")
    print("Keunggulan SVM bukan sekadar variasi acak!")
else:
    print("KESIMPULAN: Tidak signifikan (p >= 0.05).")