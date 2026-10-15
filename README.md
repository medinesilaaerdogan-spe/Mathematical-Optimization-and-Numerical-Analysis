# Mathematical-Optimization-and-Numerical-Analysis
Çok değişkenli amaç fonksiyonları, grid tabanlı optimizasyon ve 3B veri görselleştirme için Python simülasyon projesi.
import numpy as np
import pandas as pd
import plotly.graph_objects as go

def amac_fonksiyonu(x, y):
    """
    Taktik saha cihaz yerleşimi için verim (throughput) veya kapsama skoru üreten 
    çok değişkenli amaç fonksiyonu (Örnek: Kompleks bir yüzey / verim topografyası).
    """
    # Tepe noktalarında verimin maksimum, engellerde minimum olduğu matematiksel model
    r = np.sqrt(x**2 + y**2)
    z = np.sin(x / 5.0) * np.cos(y / 5.0) * 50 + (100 - r * 0.5)
    return np.maximum(0, z)

def cok_degiskenli_optimizasyon_analizi(x_araligi=(-50, 50), y_araligi=(-50, 50), adim=2.0):
    """
    Belirlenen aralıkta grid tabanlı parametre taraması (Grid Search / Optimizasyon) 
    yaparak optimum koordinatları ve verim skorlarını hesaplar.
    """
    print("--- SAYZEK Konu 28: Çok Değişkenli Matematiksel Optimizasyon Başlatılıyor ---")
    
    x_vals = np.arange(x_araligi[0], x_araligi[1], adim)
    y_vals = np.arange(y_araligi[0], y_araligi[1], adim)
    X, Y = np.meshgrid(x_vals, y_vals)
    Z = amac_fonksiyonu(X, Y)
    
    # Optimum noktayı bulma (Maksimum verim)
    max_idx = np.argmax(Z)
    optimum_x = X.flatten()[max_idx]
    optimum_y = Y.flatten()[max_idx]
    maksimum_verim = Z.flatten()[max_idx]
    
    # Rastgele / Sezgisel yerleşimlerin ortalama performansı (Karşılaştırma için)
    rastgele_orneklem_sayisi = 100
    rx = np.random.uniform(x_araligi[0], x_araligi[1], rastgele_orneklem_sayisi)
    ry = np.random.uniform(y_araligi[0], y_araligi[1], rastgele_orneklem_sayisi)
    rz = amac_fonksiyonu(rx, ry)
    ortalama_rastgele_verim = np.mean(rz)
    
    # Kazanç (Improvement) Oranı Hesabı
    kazanc_yuzdesi = ((maksimum_verim - ortalama_rastgele_verim) / ortalama_rastgele_verim) * 100
    
    print(f"Optimum X, Y Koordinatları: ({optimum_x:.2f}, {optimum_y:.2f})")
    print(f"Optimizasyon Sonrası Maksimum Verim Skoru: {maksimum_verim:.2f}")
    print(f"Rastgele Yerleşim Ortalama Verimi: {ortalama_rastgele_verim:.2f}")
    print(f"Optimizasyonun Sağladığı Yüzdesel Kazanç: %{kazanc_yuzdesi:.2f}")
    
    # Sonuçları DataFrame ile raporlama
    rapor_df = pd.DataFrame({
        "Yontem": ["Matematiksel Optimizasyon (Grid)", "Rastgele Sezgisel Yerleşim"],
        "Ortalama_Verim": [maksimum_verim, ortalama_rastgele_verim],
        "Kazanc_Orani (%)": [kazanc_yuzdesi, 0.0]
    })
    print("\n--- Performans Karşılaştırma Tablosu ---")
    print(rapor_df.to_string(index=False))
    
    return X, Y, Z, optimum_x, optimum_y

def uc_b_optimizasyon_yuzeyi_gorsellestir(X, Y, Z, opt_x, opt_y):
    """
    Optimizasyon yüzeyini ve bulunan optimum noktayı 3 boyutlu Plotly grafiği ile gösterir.
    """
    fig = go.Figure(data=[go.Surface(z=Z, x=X, y=Y, colorscale='Viridis', opacity=0.85)])
    
    opt_z = amac_fonksiyonu(opt_x, opt_y)
    
    # Optimum noktayı grafik üzerinde kırmızı işaretle göster
    fig.add_trace(go.Scatter3d(
        x=[opt_x], y=[opt_y], z=[opt_z],
        mode='markers+text',
        marker=dict(size=8, color='red'),
        text=["Optimum Nokta"],
        textposition="top center",
        name="Optimum Konum"
    ))
    
    fig.update_layout(
        title="Çok Değişkenli Amaç Fonksiyonu ve Optimum Konum Optimizasyonu",
        scene=dict(
            xaxis_title="X Parametresi",
            yaxis_title="Y Parametresi",
            zaxis_title="Sistem Verimi (Skor)",
            aspectratio=dict(x=1, y=1, z=0.7)
        ),
        margin=dict(l=0, r=0, b=0, t=40)
    )
    fig.show()

# --- Simülasyonu Çalıştırma Bloğu ---
if __name__ == "__main__":
    X, Y, Z, opt_x, opt_y = cok_degiskenli_optimizasyon_analizi()
    uc_b_optimizasyon_yuzeyi_gorsellestir(X, Y, Z, opt_x, opt_y)
import time
import matplotlib.pyplot as plt

# Farklı adım (step) değerleri listesi
adim_degerleri = [0.5, 1.0, 2.0, 5.0, 10.0]
sonuclar = []

print("--- Adım Değeri Karşılaştırma Analizi Başlatılıyor ---")
for adim in adim_degerleri:
    baslangic_zamani = time.time()
    
    # Optimizasyon analizini çalıştır
    X_temp, Y_temp, Z_temp, opt_x_temp, opt_y_temp = cok_degiskenli_optimizasyon_analizi(adim=adim)
    
    bitis_zamani = time.time()
    gecen_sure = bitis_zamani - baslangic_zamani
    maksimum_verim = amac_fonksiyonu(opt_x_temp, opt_y_temp)
    
    sonuclar.append({
        "Adım (Step)": adim,
        "Optimum X": opt_x_temp,
        "Optimum Y": opt_y_temp,
        "Maksimum Verim": maksimum_verim,
        "Süre (Saniye)": gecen_sure
    })

# Sonuçları DataFrame yapısına dönüştür ve göster
karsilastirma_df = pd.DataFrame(sonuclar)
display(karsilastirma_df)

    
