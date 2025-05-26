# Zeeman Effect Analysis Software: Technical Report

## 1. Overview

The Zeeman Effect Analysis Software is a Python-based application designed for analyzing the Zeeman effect in spectral lines. It enables precise measurement of spectral line splitting under varying magnetic field strengths, facilitating the calculation of fundamental physical constants such as the Bohr magneton and the specific charge of the electron.

The software features a user-friendly graphical interface, advanced image processing capabilities, and a sophisticated auto-detection algorithm for spectral line analysis. It provides a comprehensive workflow from image loading to results calculation, with interactive visualization tools and data export options.

## 2. Software Architecture

The application follows a modular architecture with these key components:

- **Main Window**
- **Image Processor**
- **Measurement Controller**
- **UI Manager**
- **Image Display Manager**
- **Physics Module**
- **Specialized Windows**

## 3. Key Components

### Main Window

The Main Window serves as the center of the application:

```python
def __init__(self):
    super().__init__()
    
    # Initialize state variables
    self.images = []  
    self.current_image_index = -1
    self.calibration_distance_mm = 2.0  
    self.measurements = []  
    
    # Initialize components
    self.ui_manager = UIManager(self)
    self.ui_manager.setup_layout() 
    self.image_processor = ImageProcessor()
    self.image_display_manager = ImageDisplayManager(self.image_display, self, self.ui_manager)
    self.measurement_controller = MeasurementController(self, self.ui_manager)
```

### Measurement Controller

The Measurement Controller orchestrates the measurement process:
- Manages measurement states and modes (center, inner/middle/outer ring, auto-detection)
- Processes user interactions with the image based on the current mode
- Coordinates the auto-detection workflow for spectral lines
- Handles calibration points for pixel-to-mm conversion
- Provides feedback about measurement results

```python
def set_mode(self, mode: Optional[str]):
    intended_mode = mode 

    if intended_mode == 'center':
        self.initialize_for_new_measurement()
        self.current_mode = 'center'         
    elif intended_mode and intended_mode.startswith('auto_'):
        if self.current_measurement.get('center') is None:
            QMessageBox.information(self.mw, 'Set Center First',
                                    'Please set the center point before defining an annulus for auto-detection.')
            self.current_mode = None
            self.is_defining_annulus = False
            self.auto_detect_limits = {'lower': None, 'upper': None}
            self.mw.update_display() 
            return
        else:
            self.current_mode = intended_mode
            self.is_defining_annulus = True
            self.auto_detect_limits = {'lower': None, 'upper': None} 
    else:
        self.current_mode = intended_mode
        self.is_defining_annulus = False 
        self.auto_detect_limits = {'lower': None, 'upper': None} 

    self.mw.update_display()
```

### Image Processor

The Image Processor provides image handling and analysis capabilities:
- Loads and enhances images to improve visibility of spectral lines
- Implements algorithms for detecting spectral lines
- Converts raw image data into analyzable information
- Supports the auto-detection process with sophisticated image analysis

```python
def enhance_image(self):
    if self.image is None:
        raise ValueError("No image loaded.")
    
    gray = cv2.cvtColor(self.image, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
    self.processed_image = clahe.apply(blurred)
    
    return self.processed_image

def auto_detect_radius_refined(self, processed_image, initial_center_x, initial_center_y, 
                              radius_lower_limit, radius_upper_limit, center_search_window_half_size=5):
    # Create candidate centers around initial position
    candidate_centers = []
    for dx in range(-center_search_window_half_size*2, center_search_window_half_size*2 + 1, 2):  
        for dy in range(-center_search_window_half_size*2, center_search_window_half_size*2 + 1, 2):
            candidate_centers.append((initial_center_x + dx, initial_center_y + dy))
    
    # Find optimal center and radius using weighted evaluation
    # [Additional detection code omitted for brevity]
```

### Physics Module

The Physics Module performs Zeeman effect calculations:
- Calculates angles, wavelength shifts, and energy shifts from measured radii
- Computes the Bohr magneton from a series of measurements
- Determines the specific charge of the electron
- Provides the theoretical foundation for the analysis

```python
@dataclass
class ZeemanMeasurement:
    B_field: float  # Magnetic field strength in Tesla
    wavelength: float  # Central wavelength in meters
    R_center: Optional[float] = None  # Radius of central line in mm
    R_inner: Optional[float] = None   # Radius of inner line in mm
    R_outer: Optional[float] = None   # Radius of outer line in mm
    
    # [Additional fields omitted for brevity]

def calculate_wavelength_shift(beta: float, beta_c: float, wavelength: float) -> float:
    """Calculate the wavelength shift based on the refracted angles."""
    return wavelength * (np.cos(beta_c) / np.cos(beta) - 1)

def calculate_energy_shift(delta_lambda: float, wavelength: float) -> float:
    """Calculate the energy shift based on the wavelength shift."""
    return PLANCK * LIGHT_SPEED * delta_lambda / (wavelength ** 2)
```

### UI Manager

The UI Manager constructs and organizes the user interface, it elemnates repeated code by the use of classes:

```python
def setup_layout(self):
    main_layout = QVBoxLayout(self.mw.centralWidget())
    
    # Set up navigation
    nav_layout = QHBoxLayout()
    self.mw.prev_image_btn = QPushButton("Previous Image")
    self.mw.prev_image_btn.clicked.connect(self.mw.prev_image)
    self.mw.next_image_btn = QPushButton("Next Image")
    self.mw.next_image_btn.clicked.connect(self.mw.next_image)
    
    # Set up content area with image display and control panel
    content_layout = QHBoxLayout()
    image_scroll = QScrollArea()
    image_scroll.setWidgetResizable(True)
    # [Layout setup continues...]
```

### Image Display Manager

The Image Display Manager handles rendering images:

```python
def redraw_image_with_overlays(self):
    if self.mw.current_image_index < 0 or not self.mw.images:
        return
    
    image_data = self.mw.images[self.mw.current_image_index]
    image = image_data['image'].copy()
    
    # Convert to OpenCV format for drawing
    display_cv_img = image.copy()
    
    # Draw calibration points, center, rings, etc.
    mc = self.mw.measurement_controller
    ```


## 4. Measurement Workflow

The software implemetation is as follows:

1. **Image Loading**: Load spectral line images through the file menu
2. **Magnetic Field Calibration**: Set up calibration in the calibration window
3. **Measurement Process**:
   - Set the center point of the spectral pattern
   - Either manually measure ring radii (inner, middle, outer) by clicking
   - Or use auto-detection by defining min/max radius boundaries
   - The software optimizes center points and detects precise ring radii
4. **Data Analysis**:
   - View measurements in the data table
   - Examine plots showing the relationship between field strength and energy shifts
   - Check calculated results including Bohr magneton and specific charge
5. **Data Export**:
   - Save plots as PNG/PDF
   - Export measurement data as CSV
   - Copy results to clipboard

## 5. Advanced Features

### Auto-Ring Detection Algorithm

The software implements a auto-ring detection algorithm:
- Evaluates multiple candidate center points around the manually specified center
- Uses a weighted approach considering four quality factors:
  - Distance from initial center (20%)
  - Edge strength (50%)
  - Circle completeness (20%)
  - Center distance (10%)
- Optimizes the center point and detects the precise radius even when manual specification is slightly off
- Provides detailed feedback about detection quality and adjustments

### Physics Calculations

The software performs physics calculations:
- Converts measured radii to incident and refracted angles
- Calculates wavelength shifts based on the angles
- Determines energy shifts from wavelength shifts
- Computes the Bohr magneton through linear regression of energy shifts vs. magnetic field
- Calculates the specific charge (e/m) of the electron

## 6. Usage

1. Run the application:
   ```bash
   python main.py
   ```

2. Load an image and calibrate as needed
3. Set the center point of the spectral pattern
4. Measure rings either manually or using auto-detection
5. Review measurements and calculated results
6. Export data as needed

The software provides a robust platform for Zeeman effect analysis, combining precise measurement capabilities with sophisticated physics calculations to enable accurate determination of fundamental physical constants.
