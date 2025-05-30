# Zeeman Effect Analysis Software

A Python-based application for analyzing the Zeeman effect in spectral lines.

## Overview

This software simplifies the analysis of the Zeeman effect by automating the measurement of spectral line splitting and calculating fundamental physical constants such as the Bohr magneton and specific charge of the electron.

## Features

- User-friendly graphical interface with responsive layout
- Image input from file with preview
- Advanced spectral line analysis:
  - Intelligent auto-ring detection with adaptive center point optimization
- Interactive measurement:
  - View all measurements in a table
  - Delete and redo individual measurements
  - Real-time current and magnetic field display
- Magnetic field calibration
- Calculation of Bohr magneton and specific charge
- Data visualization:
  - Plots
  - Measurement tables
  - Results summary
- Export options:
  - Save plots as PNG/PDF
  - Export data as CSV
  - Copy results to clipboard

## Installation

1. Clone this repository
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Usage

1. Activate the virtual environment:
   ```bash
   source .venv/bin/activate
   ```

2. Run the application:
   ```bash
   python main.py
   ```

3. Using the application:
   - **Load Images**: Click 'Open' to load spectral line images
   - **Calibrate**: Set up magnetic field calibration in the calibration window
   - **Take Measurements**:
     - Click to set center point
     - For manual measurement: Click to measure inner, middle, and outer radii
     - For auto-detection: Use the auto-detect buttons and define an annulus by clicking two points
     - The software will automatically optimize the center point and detect the precise ring radius
     - Review measurements in the table
     - Delete and redo measurements if needed
   - **Analyze Results**:
     - View plots in the plot window
     - Check calculated results in the results window
   - **Export Data**:
     - Save plots as PNG/PDF
     - Export measurements to CSV
     - Copy results to clipboard
