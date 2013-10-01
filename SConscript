#
# Copyright (c) 2013 by Pawel Tomulik
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE

#
# This SConscript may be used to build static libraries of StdPeriph.
#

Import(['env', 'options'])

import re
import SCons.Errors

local_opts =  [ 'MCU_TARGET', 'MCU_FAMILY', 'MCU_CORE' ]

# 
# Supported MCU targets
#
mcu_targets = [
    'STM32F10X_CL',
    'STM32F10X_LD',
    'STM32F10X_MD',
    'STM32F10X_HD',
    'STM32F10X_XL',
    'STM32F10X_LD_VL',
    'STM32F10X_MD_VL',
    'STM32F10X_HD_VL',

    # STM32F4xx
    'STM32F4XX',
    'STM32F40XX',
    'STM32F427X',
]

#
# Mapping the supported MCU targets to their families
#
mcu_family_dict = {

    # STM32F10x
    'STM32F10X_CL'      : 'STM32F10X',
    'STM32F10X_LD'      : 'STM32F10X',
    'STM32F10X_MD'      : 'STM32F10X',
    'STM32F10X_HD'      : 'STM32F10X',
    'STM32F10X_XL'      : 'STM32F10X',
    'STM32F10X_LD_VL'   : 'STM32F10X',
    'STM32F10X_MD_VL'   : 'STM32F10X',
    'STM32F10X_HD_VL'   : 'STM32F10X',

    # STM32F4xx
    'STM32F4XX'         : 'STM32F4XX',
    'STM32F40XX'        : 'STM32F4XX',
    'STM32F427X'        : 'STM32F4XX',
}

#
# Mapping MCU family to appropriate cortex core
#
mcu_core_dict = {
    'STM32F10X' : 'cortex-m3',
    'STM32F4XX' : 'cortex-m4',
}


#
# Mapping MCU family to appropriate StdPeriph library.
#
stdperiph_dict = {
    'STM32F10X' : 'STM32F10x_StdPeriph_Driver',
    'STM32F4XX' : 'STM32F4xx_StdPeriph_Driver',
}

#
# Mapping MCU family to appropriate CMSIS directory.
#
cmsis_dict = {
    'STM32F10X' : 'CMSIS/Device/ST/STM32F10x',
    'STM32F4XX' : 'CMSIS/Device/ST/STM32F4xx',
}


#
# Split 'options' into environment overrides and local options.
#
ovrr = dict([(k,v) for k,v in options.iteritems() if k not in local_opts])
opts = dict([(k,v) for k,v in options.iteritems() if k in local_opts])

#
# MCU_TARGET
#
try:             mcu_target = opts['MCU_TARGET']
except KeyError: raise SCons.Errors.UserError('MCU_TARGET not specified')
if not mcu_target.upper() in mcu_targets:
    raise SCons.Errors.UserError('unsupported MCU_TARGET: %s' % mcu_target)
mcu_target = mcu_target.upper()

#
# MCU_FAMILY
#
mcu_family = opts.get('MCU_FAMILY', mcu_family_dict[mcu_target])
mcu_family = mcu_family.upper()

#
# STDPERIPH_DIR
#
stdperiph_dir = stdperiph_dict[mcu_family]

#
# CMSIS_DIR 
#
cmsis_dir = cmsis_dict[mcu_family]

#
# MCU_CORE
#
mcu_core = opts.get('MCU_CORE', mcu_core_dict[mcu_family])
mcu_core = mcu_core.lower()
if mcu_core:    mcu_flags = ['-mcpu=%s' % mcu_core, '-mthumb']
else:           mcu_flags = []

#
# CPPDEFINES
#
cppdefs = ovrr.get('CPPDEFINES', env.get('CPPDEFINES',[]))
if 'USE_STDPERIPH_DRIVER' not in cppdefs: cppdefs.append('USE_STDPERIPH_DRIVER')
if mcu_target not in cppdefs:             cppdefs.append(mcu_target)
ovrr['CPPDEFINES'] = cppdefs

#
# CPPPATH
#
subdirs = ['inc', 'tpl', '%s/Include' % cmsis_dir ]
cpppath = ovrr.get('CPPPATH', env.get('CPPPATH',[]))
cpppath += [ '%s/%s' % (stdperiph_dir, s) for s in subdirs ]
cpppath += ['#CMSIS/Include']
ovrr['CPPPATH'] = cpppath

#
# CFLAGS
#
cflags = ovrr.get('CFLAGS', env.get('CFLAGS', []))
cflags += mcu_flags
cflags += ["-Wa,-adhlns='${TARGET}.lst'"]
ovrr['CFLAGS'] = cflags

#
# CXXFLAGS
#
cxxflags = ovrr.get('CXXFLAGS', ovrr.get('CXXFLAGS', []))
cxxflags += mcu_flags
cxxflags += ["-Wa,-adhlns='${TARGET}.lst'"]
ovrr['CXXFLAGS'] = cxxflags

#
# LINKFLAGS
#
linkflags = ovrr.get('LINKFLAGS', ovrr.get('LINKFLAGS', []))
linkflags += mcu_flags
ovrr['LINKFLAGS'] = linkflags

#
# SOURCES
#
subdirs = ['src', 'tpl']
sources = [ env.Glob('%s/%s/*.c' % (stdperiph_dir, s)) for s in subdirs ]
sources = Flatten(sources)

#
# LIBRARY NAME
#
libname = "stdperiph_%s" % mcu_target.lower()

#
# BUILD THE LIBRARY
#
target = env.Library(libname, sources, **ovrr)

#
# SIDE EFFECTS
#
lst_files = [re.sub(r'\.c$', '.o.lst', s.get_path()) for s in sources]
env.SideEffect(lst_files, target)

#
# RETURN WHAT WAS BUILT
#
Return('target')

# Local Variables:
# # tab-width:4
# # indent-tabs-mode:nil
# # End:
# vim: set syntax=scons expandtab tabstop=4 shiftwidth=4:
