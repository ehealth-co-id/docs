# API reservasi-backend

---

## 1. BaseUrl

```
https://{domain-anda}/
```

> **Contoh**
> `https://api.demo.godentist.co.id/`

> **Catatan:** Semua endpoint bersifat *public* — **tidak** memerlukan autentikasi.

---

## 2. Endpoint FHIR Discovery

> *FHIR resources* tersedia di path `/fhir/{resourceType}`.
> Pada API ini:
>
> * **HealthcareService** me­-representasikan jenis layanan (misal: Poli Umum, Poli Gigi).
> * **Schedule** me­-representasikan pilihan tanggal ketersediaan.
> * **Slot** me­-representasikan pilihan jam pada tanggal terpilih.

### 2.1. Daftar HealthcareService Aktif

* **Endpoint**

  ```
  GET /fhir/HealthcareService/?active=true
  ```
* **Deskripsi**
  Mengembalikan *Bundle* `HealthcareService` dengan `active=true`.
* **Contoh Request**

  ```http
  GET https://api.demo.godentist.co.id/fhir/HealthcareService/?active=true
  ```
* **Contoh Response (200 OK)**

  ```json
  {
    "resourceType": "Bundle",
    "entry": [
      {
        "resource": {
          "resourceType": "HealthcareService",
          "id": "4745827",
          "name": "Poli Umum",
          "active": true,
          …
        }
      },
      {
        "resource": {
          "resourceType": "HealthcareService",
          "id": "4745828",
          "name": "Poli Gigi",
          "active": true,
          …
        }
      }
    ]
  }
  ```

---

### 2.2. Daftar Schedule untuk HealthcareService

* **Endpoint**

  ```
  GET /fhir/Schedule/
    ?active=true
    &date=ge{iso8601Date}
    &actor={healthcareServiceId}
    &_count={maxResults}
  ```
* **Deskripsi**
  Mengembalikan *Bundle* `Schedule` (pilihan tanggal) untuk `HealthcareService` tertentu.
* **Contoh Request**

  ```http
  GET https://api.demo.godentist.co.id/fhir/Schedule/
      ?active=true
      &date=ge2025-07-08T00:00:00Z
      &actor=4745827
      &_count=100
  ```
* **Contoh Response (200 OK)**

  ```json
  {
    "resourceType": "Bundle",
    "entry": [
      {
        "resource": {
          "resourceType": "Schedule",
          "id": "6505052",
          "active": true,
          "planningHorizon": {
            "start": "2025-07-08T00:00:00+07:00",
            "end":   "2025-07-08T23:59:59+07:00"
          },
          …
        }
      },
      …
    ]
  }
  ```

---

### 2.3. Daftar Slot untuk Schedule

* **Endpoint**

  ```
  GET /fhir/Slot/?schedule={scheduleId}&_count={maxResults}
  ```
* **Deskripsi**
  Mengembalikan *Bundle* `Slot` (pilihan jam) untuk `Schedule` yang dipilih.
* **Contoh Request**

  ```http
  GET https://api.demo.godentist.co.id/fhir/Slot/
      ?schedule=6505052
      &_count=100
  ```
* **Contoh Response (200 OK)**

  ```json
  {
    "resourceType": "Bundle",
    "entry": [
      {
        "resource": {
          "resourceType": "Slot",
          "id": "6505053",
          "start": "2025-07-08T10:00:00+07:00",
          "end":   "2025-07-08T10:15:00+07:00",
          …
        }
      },
      …
    ]
  }
  ```

---

## 3. Memulai Reservasi

### 3.1. POST `/reservasi/start`

* **Endpoint**

  ```
  POST /reservasi/start
  ```

* **Deskripsi**
  Membuat `Appointment` FHIR baru (men-”book” slot).

* **Request Headers**

  ```http
  Content-Type: application/json
  ```

* **Body Request**

  ```json
  {
    "resourceType": "Appointment",
    "meta": {
      "extension": [
        {
          "url": "https://ehealth.co.id/terminology/initiator-component",
          "valueString": "godentist-app"
        }
      ]
    },
    "extension": [
      {
        "url": "https://ehealth.co.id/terminology/appointment-encounter-class",
        "valueCoding": {
          "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
          "code": "AMB",
          "display": "ambulatory"
        }
      },
      {
        "url": "https://ehealth.co.id/terminology/patient-name",
        "valueString": "Pasien Test"
      },
      {
        "url": "https://ehealth.co.id/terminology/patient-phone",
        "valueString": "080000000000"
      },
      {
        "url": "https://ehealth.co.id/terminology/patient-birth-date",
        "valueString": "07/06/2024"
      },
      {
        "url": "https://ehealth.co.id/terminology/patient-nik",
        "valueString": "0000000000000000"
      },
      {
        "url": "https://ehealth.co.id/terminology/patient-email",
        "valueString": "test@example.com"
      },
      {
        "url": "https://ehealth.co.id/terminology/practitioner-not-set",
        "valueBoolean": true
      }
    ],
    "status": "booked",
    "participant": [
      {
        "actor": { "reference": "HealthcareService/4745827" },
        "status": "accepted"
      },
      {
        "actor": { "reference": "Location/godentist-demo" },
        "status": "accepted"
      }
    ],
    "serviceCategory": {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/service-category",
          "code": "10",
          "display": "Dental"
        },
        {
          "system": "https://ehealth.co.id/terminology/service-category",
          "code": "dental",
          "display": "Dental"
        }
      ],
      "text": "Dental"
    },
    "serviceType": [
      {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/service-type",
            "code": "88",
            "display": "General Dental"
          },
          {
            "system": "https://ehealth.co.id/terminology/service-type",
            "code": "dental",
            "display": "Dental"
          }
        ],
        "text": "General Dental"
      }
    ],
    "appointmentType": {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/v2-0276",
          "code": "WALKIN",
          "display": "A previously unscheduled walk-in visit"
        }
      ],
      "text": "walk‑in"
    },
    "start": "2025-07-08T10:00:00+07:00",
    "slot": [
      { "reference": "Slot/6505053" }
    ]
  }
  ```

* **Penjelasan `serviceCategory` & `serviceType`**
  Nilai untuk `serviceCategory` dan `serviceType` **diambil langsung** dari data `HealthcareService` yang dipanggil di endpoint `/fhir/HealthcareService`.

  * `serviceCategory`: kategori besar layanan (misal: kode “10” → Dental)
  * `serviceType`    : tipe spesifik layanan (misal: kode “88” → General Dental)

* **Contoh Response (201 Created)**

  ```json
  {
    "resourceType": "Appointment",
    "id": "123456",
    "status": "booked",
    "start": "2025-07-08T10:00:00+07:00",
    …
  }
  ```

---

## 4. Penanganan Kesalahan

| HTTP Status      | Keterangan               | Penyebab Umum                         |
| ---------------- | ------------------------ | ------------------------------------- |
| 400 Bad Request  | Parameter tidak valid    | Format JSON salah, field wajib hilang |
| 404 Not Found    | Resource tidak ditemukan | ID Schedule/Slot/Service salah        |
| 409 Conflict     | Slot sudah dibooking     | Double‑booking                        |
| 500 Server Error | Kesalahan server         | Periksa log server                    |

---

## 5. Best Practices

1. **Cek Ketersediaan**

   * Panggil `/fhir/Schedule` → pilih tanggal.
   * Panggil `/fhir/Slot`     → pilih jam.
2. **Validasi Data**

   * Pastikan format tanggal (ISO‑8601) dan string NIK/email valid.
3. **Penanganan Pagination**

   * Gunakan parameter `_count` dan link pada Bundle untuk hasil banyak.
4. **Pengujian**

   * Uji dengan Postman/Insomnia tanpa header Auth.
   * Simulasikan double‑booking untuk cek `409 Conflict`.
