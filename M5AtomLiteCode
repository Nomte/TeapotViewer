from machine import I2C, Pin
import sh1106
import mpu6050
import math
from time import sleep

i2c = I2C(scl=Pin(22), sda=Pin(21))
display = sh1106.SH1106_I2C(128, 64, i2c)
accel = mpu6050.accel(i2c)

def get_accel(samples=10, calibration=None):
    # Setup a dict of measure at 0
    result = {}
    for _ in range(samples):
        v = accel.get_values()

        for m in v.keys():
            # Add on value / samples (to generate an average)
            result[m] = result.get(m, 0) + v[m] / samples

    if calibration: 
        # Remove calibration adjustment
        for m in calibration.keys():
            result[m] -= calibration[m]

    return result

def calibrate(threshold=50):
    print('Calibrating...', end='')
    while True:
        v1 = get_accel(100)
        v2 = get_accel(100)
        if all(abs(v1[m] - v2[m]) < threshold for m in v1.keys()):
            print('Done.')
            return v1
        
class Point3D:
    def __init__(self, x = 0, y = 0, z = 0):
        self.x, self.y, self.z = x, y, z

    def rotateX(self, deg):
        """ Rotates this point around the X axis the given number of degrees. """
        rad = deg * math.pi / 180
        cosa = math.cos(rad)
        sina = math.sin(rad)
        y = self.y * cosa - self.z * sina
        z = self.y * sina + self.z * cosa
        return Point3D(self.x, y, z)

    def rotateY(self, deg):
        """ Rotates this point around the Y axis the given number of degrees. """
        rad = deg * math.pi / 180
        cosa = math.cos(rad)
        sina = math.sin(rad)
        z = self.z * cosa - self.x * sina
        x = self.z * sina + self.x * cosa
        return Point3D(x, self.y, z)

    def rotateZ(self, deg):
        """ Rotates this point around the Z axis the given number of degrees. """
        rad = deg * math.pi / 180
        cosa = math.cos(rad)
        sina = math.sin(rad)
        x = self.x * cosa - self.y * sina
        y = self.x * sina + self.y * cosa
        return Point3D(x, y, self.z)

    def project(self, win_width, win_height, fov, viewer_distance):
        """ Transforms this 3D point to 2D using a perspective projection. """
        factor = fov / (viewer_distance + self.z)
        x = self.x * factor + win_width / 2
        y = -self.y * factor + win_height / 2
        return Point3D(x, y, self.z)

class Simulation:
    def __init__(self, width=128, height=64, fov=128, distance=1024):

        self.vertices = [
            Point3D(0,0,0),
            Point3D(178,142,0),
            Point3D(126,142,127),
            Point3D(0,142,179),
            Point3D(-126,142,127),
            Point3D(-178,142,0),
            Point3D(-126,142,-126),
            Point3D(0,142,-178),
            Point3D(126,142,-126),
            Point3D(219,49,0),
            Point3D(238,-36,0),
            Point3D(155,49,156),
            Point3D(169,-36,169),
            Point3D(0,49,220),
            Point3D(0,-36,238),
            Point3D(-155,49,156),
            Point3D(-169,-36,169),
            Point3D(-219,49,0),
            Point3D(-238,-36,0),
            Point3D(-155,49,-155),
            Point3D(-169,-36,-168),
            Point3D(0,49,-219),
            Point3D(0,-36,-237),
            Point3D(155,49,-155),
            Point3D(169,-36,-168),
            Point3D(208,-97,0),
            Point3D(148,-97,148),
            Point3D(0,-97,209),
            Point3D(-148,-97,148),
            Point3D(-208,-97,0),
            Point3D(-148,-97,-147),
            Point3D(0,-97,-207),
            Point3D(148,-97,-147),
            Point3D(152,-137,0),
            Point3D(108,-137,109),
            Point3D(0,-137,153),
            Point3D(-108,-137,109),
            Point3D(-152,-137,0),
            Point3D(-108,-137,-108),
            Point3D(0,-137,-152),
            Point3D(108,-137,-108),
            Point3D(-190,97,0),
            Point3D(-287,94,0),
            Point3D(-321,70,0),
            Point3D(-184,111,27),
            Point3D(-300,106,27),
            Point3D(-339,70,27),
            Point3D(-178,124,0),
            Point3D(-312,117,0),
            Point3D(-357,70,0),
            Point3D(-184,111,-26),
            Point3D(-300,106,-26),
            Point3D(-339,70,-26),
            Point3D(-302,17,0),
            Point3D(-238,-36,0),
            Point3D(-313,5,27),
            Point3D(-232,-54,27),
            Point3D(-325,-5,0),
            Point3D(-226,-71,0),
            Point3D(-313,5,-26),
            Point3D(-232,-54,-26),
            Point3D(202,26,0),
            Point3D(284,70,0),
            Point3D(321,142,0),
            Point3D(202,-22,59),
            Point3D(302,49,41),
            Point3D(357,142,22),
            Point3D(202,-71,0),
            Point3D(320,28,0),
            Point3D(393,142,0),
            Point3D(202,-22,-58),
            Point3D(302,49,-40),
            Point3D(357,142,-21),
            Point3D(0,231,0),
            Point3D(27,211,28),
            Point3D(16,178,17),
            Point3D(-27,211,28),
            Point3D(-16,178,17),
            Point3D(-27,211,-26),
            Point3D(-16,178,-16),
            Point3D(27,211,-26),
            Point3D(16,178,-16),
            Point3D(154,142,0),
            Point3D(109,142,110),
            Point3D(0,142,155),
            Point3D(-109,142,110),
            Point3D(-154,142,0),
            Point3D(-109,142,-109),
            Point3D(0,142,-154),
            Point3D(109,142,-109),
        ]

        # Define the edges, the numbers are indices to the vertices above.
        self.edges  = [
            (1,2),(2,3),(3,4),(4,5),(5,6),(6,7),(7,8),(1,8),(2,11),
            (9,11),(1,9),(11,12),(10,12),(9,10),(3,13),(11,13),(13,14),
            (12,14),(4,15),(13,15),(15,16),(14,16),(5,17),(15,17),
            (17,18),(16,18),(6,19),(17,19),(19,20),(18,20),(7,21),
            (19,21),(21,22),(20,22),(8,23),(21,23),(23,24),(22,24),
            (9,23),(10,24),(12,26),(25,26),(10,25),(26,34),(25,33),
            (14,27),(26,27),(27,35),(16,28),(27,28),(28,36),(18,29),
            (28,29),(29,37),(20,30),(29,30),(30,38),(22,31),(30,31),
            (31,39),(24,32),(31,32),(32,40),(25,32),(33,34),(34,35),
            (35,36),(36,37),(37,38),(38,39),(39,40),(33,40),(41,44),
            (44,45),(42,45),(41,42),(45,46),(43,46),(42,43),(44,47),
            (47,48),(45,48),(48,49),(46,49),(47,50),(50,51),(48,51),
            (51,52),(49,52),(41,50),(42,51),(43,52),(46,55),(53,55),
            (43,53),(55,56),(54,56),(53,54),(49,57),(55,57),(57,58),
            (56,58),(52,59),(57,59),(59,60),(58,60),(53,59),(54,60),
            (61,64),(64,65),(62,65),(61,62),(65,66),(63,66),(62,63),
            (64,67),(67,68),(65,68),(68,69),(66,69),(67,70),(70,71),
            (68,71),(71,72),(69,72),(61,70),(62,71),(63,72),(74,80),
            (73,74),(74,75),(75,81),(74,76),(75,77),(73,76),(76,77),
            (76,78),(77,79),(73,78),(78,79),(78,80),(79,81),(73,80),
            (80,81),(75,83),(82,83),(83,84),(77,85),(84,85),(85,86),
            (79,87),(86,87),(87,88),(81,89),(88,89),(82,89),
        ]

        # Dimensions
        self.projection = [width, height, fov, distance]

    def run(self):
        # Starting angle (unrotated in any dimension)
        angleX, angleY, angleZ = 0, 0, 0

        calibration = calibrate()

        while 1:

            data = get_accel(10, calibration)

            angleX = data['AcX'] / 128
            angleY = data['AcY'] / 128

            t = []
            for v in self.vertices:
                # Rotate the point around X axis, then around Y axis, and finally around Z axis.
                r = v.rotateX(angleX).rotateY(angleY).rotateZ(angleZ)

                # Transform the point from 3D to 2D
                p = r.project(*self.projection)

                # Put the point in the list of transformed vertices
                t.append(p)

            display.fill(0)

            for e in self.edges:
                display.line(*to_int(t[e[0]].x, t[e[0]].y, t[e[1]].x, t[e[1]].y, 1))

            display.show()

def to_int(*args):
    return [int(v) for v in args]

#============START============#
display.fill(0)
display.rotate(True)
display.text("This is",36,12)
display.text("a Teapot",32,28)
display.text("renderer.",32,44)
display.show()
sleep(4.2)

s = Simulation()
s.run()
