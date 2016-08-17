## .framework
```shell
# Sets the target folders and the final framework product.
FMK_NAME=${PROJECT_NAME}
# Install dir will be the final output to the framework.
# The following line create it in the root folder of the current project.
INSTALL_DIR=${SRCROOT}/../${FMK_NAME}Products/${FMK_NAME}.framework
# Working dir will be deleted after the framework creation.
WRK_DIR=build
DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework
# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
echo "${DEVICE_DIR}"
echo "${SIMULATOR_DIR}"
echo "${BUILD_ROOT}"
echo |pwd
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build
# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"
# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
 lipo -create "${DEVICE_DIR}/${FMK_NAME}" "${SIMULATOR_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"

 rm -r "${WRK_DIR}"
 open "${INSTALL_DIR}/.."
```

## .a
```shell
# Sets the target folders and the final framework product.
# FMK_NAME=${PROJECT_NAME}
FMK_NAME="SSEApiUI"
LIB_NAME=lib${FMK_NAME}.a
# Install dir will be the final output to the framework.
# The following line create it in the root folder of the current project.
INSTALL_DIR=${SRCROOT}/../${FMK_NAME}Products
# Working dir will be deleted after the framework creation.
WRK_DIR=build
DEVICE_DIR=${WRK_DIR}/Release-iphoneos
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator

# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
echo "${DEVICE_DIR}"
echo "${SIMULATOR_DIR}"
echo "${BUILD_ROOT}"
echo |pwd
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build
# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}/include/"
# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
lipo -create "${DEVICE_DIR}/lib${FMK_NAME}.a" "${SIMULATOR_DIR}/lib${FMK_NAME}.a" -output "${INSTALL_DIR}/libSSEApi.a"

# copy headers

cp -R "${DEVICE_DIR}/include/${FMK_NAME}" "${INSTALL_DIR}/include/${FMK_NAME}"
rm -r "${WRK_DIR}"
open "${INSTALL_DIR}/.."
#open "${DEVICE_DIR}/.."

```
