# python-3
# ─── Dekoratorlar amaliyoti ──────────────────────────────────────────────
import time
import random
from functools import wraps, lru_cache

# 1) Eng oddiy timer
def timer(funk):
    @wraps(funk)
    def wrapper(*args, **kwargs):
        boshlanish = time.perf_counter()
        natija = funk(*args, **kwargs)
        tugadi = time.perf_counter() - boshlanish
        print(f"⏱  {funk.__name__:<20} {tugadi*1000:>8.2f} ms")
        return natija
    return wrapper

@timer
def sekin(n):
    return sum(i * i for i in range(n))

sekin(500_000)
print(sekin.__name__)   # "sekin" — wraps tufayli

# 2) lru_cache — fibonacci tezligi
@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))    # bir lahzada

# 3) Parametrli retry
def retry(marotaba=3, kutish=0.1):
    def dek(funk):
        @wraps(funk)
        def wrapper(*args, **kwargs):
            oxirgi_xato = None
            for u in range(marotaba):
                try:
                    return funk(*args, **kwargs)
                except Exception as e:
                    oxirgi_xato = e
                    print(f"  urinish {u+1}/{marotaba} ❌ {e}")
                    time.sleep(kutish)
            raise oxirgi_xato
        return wrapper
    return dek

@retry(marotaba=5, kutish=0.05)
def shubhali_api():
    if random.random() < 0.6:
        raise ConnectionError("network down")
    return {"ok": True}

print(shubhali_api())

# 4) Authorization dekoratori — chaqiruvni bloklaydi
def faqat_admin(funk):
    @wraps(funk)
    def wrapper(foydalanuvchi, *args, **kwargs):
        if foydalanuvchi.get("rol") != "admin":
            raise PermissionError("Faqat admin uchun")
        return funk(foydalanuvchi, *args, **kwargs)
    return wrapper

@faqat_admin
def o_chirish(foydalanuvchi, fayl):
    print(f"{fayl} o'chirildi")

o_chirish({"rol": "admin"}, "/tmp/x")
# o_chirish({"rol": "user"}, "/tmp/x")     # PermissionError

# 5) Dekoratorlar stacki — tartib muhim
@timer
@retry(marotaba=3)
def ko_p_qatlamli():
    if random.random() < 0.5:
        raise ValueError("ko'p urinish kerak")
    return 42

print(ko_p_qatlamli())

# 6) cached_property — class atributi sifatida lazy hisob
class Hisoblovchi:
    def __init__(self, sonlar):
        self.sonlar = sonlar

    @property
    def jami(self):                       # har murojaatda qayta hisoblanadi
        print("(jami hisoblanmoqda)")
        return sum(self.sonlar)

h = Hisoblovchi([1, 2, 3, 4, 5])
print(h.jami)
print(h.jami)                              # qayta hisoblanadi
