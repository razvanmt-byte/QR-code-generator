import qrcode
import numpy as np
import trimesh
from trimesh.creation import box
from trimesh.scene import Scene

# Generate QR code image
qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H)
qr.add_data('https://linktr.ee/Mititelu_Razvan')
qr.make()
matrix = np.array(qr.get_matrix(), dtype=int)

# Constants
qr_size = 25.0  # mm
base_thickness = 2.0  # mm
module_thickness = 1.0  # mm
module_size = qr_size / matrix.shape[0]  # mm per module

# Create the white base plate
base_plate = box(extents=[qr_size, qr_size, base_thickness])
base_plate.apply_translation([qr_size / 2, qr_size / 2, base_thickness / 2])
base_plate.visual.face_colors = [255, 255, 255, 255]  # White

# Create black QR code modules
black_modules = []
rows, cols = matrix.shape
for y in range(rows):
    for x in range(cols):
        if matrix[y, x] == 1:
            block = box(extents=[module_size, module_size, module_thickness])
            block.apply_translation([
                (x + 0.5) * module_size,
                (rows - y - 0.5) * module_size,
                base_thickness + module_thickness / 2
            ])
            block.visual.face_colors = [0, 0, 0, 255]  # Black
            black_modules.append(block)

# Combine black modules into one mesh
qr_code_mesh = trimesh.util.concatenate(black_modules)

# Create scene with two separate geometries
scene = Scene()
scene.add_geometry(base_plate, node_name='White_Base')
scene.add_geometry(qr_code_mesh, node_name='Black_QR_Modules')

# Export as a single OBJ file with two objects
output_path = "/mnt/data/qr_code_plate.obj"
scene.export(output_path)

output_path
