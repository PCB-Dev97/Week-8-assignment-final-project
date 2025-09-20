# Week-8-assignment-final-project

-- clinic_booking.sql

-- 1. Create database
CREATE DATABASE IF NOT EXISTS clinic_booking
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE clinic_booking;

-- 2. Tables

-- Patients table: stores patient personal info
CREATE TABLE Patients (
    patient_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender ENUM('Male','Female','Other') NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- Doctors table: store doctor info
CREATE TABLE Doctors (
    doctor_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    specialization VARCHAR(100) NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- One-to-One example: each doctor has exactly one profile with more details

CREATE TABLE DoctorProfiles (
    doctor_id INT PRIMARY KEY,
    biography TEXT,
    years_of_experience INT,
    clinic_address VARCHAR(255),
    -- doctor_id references Doctors.doctor_id, enforcing 1-to-1
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Clinics table: multiple clinics/hospitals
CREATE TABLE Clinics (
    clinic_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(255),
    UNIQUE (name, address)
) ENGINE=InnoDB;

-- A doctor may work at multiple clinics; a clinic has many doctors
-- Many-to-Many relationship between Doctors and Clinics

CREATE TABLE DoctorClinic (
    doctor_id INT NOT NULL,
    clinic_id INT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    PRIMARY KEY (doctor_id, clinic_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE,
    FOREIGN KEY (clinic_id) REFERENCES Clinics(clinic_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Appointments table: a patient books an appointment with a doctor at a clinic

CREATE TABLE Appointments (
    appointment_id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    clinic_id INT NOT NULL,
    appointment_date DATETIME NOT NULL,
    duration_minutes INT NOT NULL,
    status ENUM('Scheduled','Completed','Cancelled','No-Show') NOT NULL DEFAULT 'Scheduled',
    notes TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Foreign keys
    FOREIGN KEY (patient_id) REFERENCES Patients(patient_id)
      ON DELETE RESTRICT
      ON UPDATE CASCADE,
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
      ON DELETE RESTRICT
      ON UPDATE CASCADE,
    FOREIGN KEY (clinic_id) REFERENCES Clinics(clinic_id)
      ON DELETE RESTRICT
      ON UPDATE CASCADE,
    
    -- To avoid double booking: same doctor, same clinic, overlapping times
    -- (Some constraints / application logic needed; simplest uniqueness on doctor + appointment_date)
    UNIQUE KEY ux_doctor_clinic_datetime (doctor_id, clinic_id, appointment_date)
) ENGINE=InnoDB;

-- Services table: different types of services offered (e.g. “General Consultation”, “Dental Cleaning”, etc.)

CREATE TABLE Services (
    service_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    duration_minutes INT NOT NULL,
    price DECIMAL(10,2) NOT NULL
) ENGINE=InnoDB;

-- Many-to-Many between Appointments and Services: one appointment can have multiple services; one service can be used in many appointments

CREATE TABLE AppointmentServices (
    appointment_id INT NOT NULL,
    service_id INT NOT NULL,
    PRIMARY KEY (appointment_id, service_id),
    FOREIGN KEY (appointment_id) REFERENCES Appointments(appointment_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE,
    FOREIGN KEY (service_id) REFERENCES Services(service_id)
      ON DELETE RESTRICT
      ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Billing / Payments table: each appointment has a bill; optionally payments

CREATE TABLE Bills (
    bill_id INT AUTO_INCREMENT PRIMARY KEY,
    appointment_id INT NOT NULL UNIQUE,  -- 1-to-1 with appointment
    amount DECIMAL(10,2) NOT NULL,
    billing_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    paid BOOLEAN DEFAULT FALSE,
    payment_date DATETIME,
    FOREIGN KEY (appointment_id) REFERENCES Appointments(appointment_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Example: Users for system login (doctor, admin, receptionist) — roles

CREATE TABLE Roles (
    role_id INT AUTO_INCREMENT PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(200),
    email VARCHAR(255) UNIQUE,
    role_id INT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES Roles(role_id)
      ON DELETE RESTRICT
      ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Logs: audit trail

CREATE TABLE AppointmentLogs (
    log_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    appointment_id INT NOT NULL,
    changed_by_user_id INT NOT NULL,
    change_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    change_description TEXT NOT NULL,
    FOREIGN KEY (appointment_id) REFERENCES Appointments(appointment_id)
      ON DELETE CASCADE
      ON UPDATE CASCADE,
    FOREIGN KEY (changed_by_user_id) REFERENCES Users(user_id)
      ON DELETE SET NULL
      ON UPDATE CASCADE
) ENGINE=InnoDB;

