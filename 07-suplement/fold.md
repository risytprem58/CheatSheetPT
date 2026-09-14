# =========================================================
# UJI SIGNIFIKANSI STATISTIK (ANTI-CRASH & AUTO-SAVE)
# =========================================================

import os
import pandas as pd
import numpy as np
from scipy import stats
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import Pipeline
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier

output_stat_path = "../6. Output/uji_signifikansi_svm_knn.xlsx"
os.makedirs(os.path.dirname(output_stat_path), exist_ok=True)

# CEK APAKAH SUDAH PERNAH DIHITUNG SEBELUMNYA
if os.path.exists(output_stat_path):
    print("✔ File hasil ditemukan! Memuat data tersimpan tanpa perlu hitung ulang...")
    df_res = pd.read_excel(output_stat_path)
    print("\n" + "="*55)
    print(df_res.to_string(index=False))
    print("="*55)
else:
    print("Menjalankan pengujian 5-fold CV untuk SVM dan KNN...")

    # 1. Pipeline SVM Terbaik (Rank 1)
    pipe_svm = Pipeline([
        ('preprocessing', preprocessor),
        ('model', SVC(kernel='poly', C=10, gamma='scale', degree=3, class_weight='balanced', random_state=42))
    ])

    # 2. Pipeline KNN Terbaik (Rank 8)
    pipe_knn = Pipeline([
        ('preprocessing', preprocessor),
        ('model', KNeighborsClassifier(n_neighbors=3, weights='distance', metric='manhattan'))
    ])

    # 3. Hitung F1-score lintas 5 fold
    scores_svm = cross_val_score(pipe_svm, X, y_encoded, cv=cv, scoring='f1_macro', n_jobs=-1)
    scores_knn = cross_val_score(pipe_knn, X, y_encoded, cv=cv, scoring='f1_macro', n_jobs=-1)

    # 4. Uji Signifikansi Statistik
    t_stat, p_val_ttest = stats.ttest_rel(scores_svm, scores_knn)
    w_stat, p_val_wilcoxon = stats.wilcoxon(scores_svm, scores_knn)

    # 5. SIMPAN HASIL KE EXCEL AGAR AMAN DARI CRASH
    data_hasil = []
    for fold_idx in range(len(scores_svm)):
        data_hasil.append({
            'Fold': f'Fold {fold_idx+1}',
            'F1 SVM (Rank 1)': round(scores_svm[fold_idx], 4),
            'F1 KNN (Rank 8)': round(scores_knn[fold_idx], 4),
            'Selisih (SVM - KNN)': round(scores_svm[fold_idx] - scores_knn[fold_idx], 4)
        })

    df_res = pd.DataFrame(data_hasil)
    
    # Tambahkan baris ringkasan statistik
    df_summary = pd.DataFrame([{
        'Fold': 'RATA-RATA (MEAN)',
        'F1 SVM (Rank 1)': round(scores_svm.mean(), 4),
        'F1 KNN (Rank 8)': round(scores_knn.mean(), 4),
        'Selisih (SVM - KNN)': round(scores_svm.mean() - scores_knn.mean(), 4)
    }, {
        'Fold': 'P-VALUE (t-test)',
        'F1 SVM (Rank 1)': f"t = {round(t_stat, 4)}",
        'F1 KNN (Rank 8)': f"p-value = {p_val_ttest:.6f}",
        'Selisih (SVM - KNN)': 'SIGNIFIKAN (p < 0.05)' if p_val_ttest < 0.05 else 'TIDAK SIGNIFIKAN'
    }, {
        'Fold': 'P-VALUE (Wilcoxon)',
        'F1 SVM (Rank 1)': f"W = {round(w_stat, 4)}",
        'F1 KNN (Rank 8)': f"p-value = {p_val_wilcoxon:.6f}",
        'Selisih (SVM - KNN)': 'SIGNIFIKAN (p < 0.05)' if p_val_wilcoxon < 0.05 else 'TIDAK SIGNIFIKAN'
    }])

    df_final = pd.concat([df_res, df_summary], ignore_index=True)
    df_final.to_excel(output_stat_path, index=False)
    print(f"✔ Berhasil disimpan ke: {output_stat_path}")
    print("\n" + "="*55)
    print(df_final.to_string(index=False))
    print("="*55)