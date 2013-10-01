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

Import(['env', 'overrides'])

import re

local_opts =  [
    'MCU_CORE',   # optional
    'MCU_FAMILY', # e.g. 'stm32f10x', 'stm32f4xx'
    'MCU_LINE',   # e.g. 'ld', 'vl', 'md', 'md-vl', etc...
]

mcu_families = {
    'STM32F10X',
    'STM32F4XX',
}

mcu_cores = {
    'STM32F10X' : 'cortex-m3',
    'STM32F4XX' : 'cortex-m4',
}

mcu_lines = { 
    'STM32F10X' : {
        'CL'    : 'Connectivity Line', 
        'LD'    : 'Low Density Performance Line', 
        'MD'    : 'Medium Density Performance Line', 
        'HD'    : 'High Density Performance Line', 
        'XL'    : 'XL-Density Line', 
        'LD_VL' : 'Low Density Value Line', 
        'MD_VL' : 'Medium Density Value Line', 
        'HD_VL' : 'High Density Value Line'
    },
    'STM32F4XX' : {
    }
}

stdperiph_dirs = {
    'STM32F10X' : 'STM32F10x_StdPeriph_Driver',
    'STM32F4XX' : 'STM32F4xx_StdPeriph_Driver',
}


# Keyword arguments to Library builder.
kwargs  = dict([(k,v) for k,v in overrides.iteritems() if k not in local_opts])
# Other options passed via 'overrides'.
opts = dict([(k,v) for k,v in overrides.iteritems() if k in local_opts])

#
# MCU_FAMILY
#
try:              mcu_family = opts['MCU_FAMILY']
except KeyError:  raise SCons.Errors.UserError('MCU_FAMILY not specified!')
if not mcu_family.upper() in mcu_families:
    raise SCons.Error.UserError('unsupported MCU_FAMILY: %s' % mcu_family)

stdperiph_dir = stdperiph_dirs[mcu_family.upper()]

#
# MCU_LINE
#
try:              mcu_line = opts['MCU_LINE']
except KeyError:  raise SCons.Errors.UserError('MCU_LINE not specified!')
mcu_line  = re.sub('\W', '_', mcu_line)
if not mcu_line.upper() in mcu_lines[mcu_family.upper()]:
    raise SCons.Error.UserError('unsupported MCU_LINE: %s' % mcu_line)

#
# MCU_CORE
#
mcu_core = opts.get('MCU_CORE', mcu_cores[mcu_family.upper()])
if mcu_core:    mcu_flags = ['-mcpu=%s' % mcu_core, '-mthumb']
else:           mcu_flags = []

#
# CPPDEFINES
#
cppdefs = env.get('CPPDEFINES',[]) + kwargs.get('CPPDEFINES',[])
if not 'USE_STDPERIPH_DRIVER' in cppdefs:
    cppdefs.append('USE_STDPERIPH_DRIVER')
cppdefs.append('%s_%s' % (mcu_family.upper(), mcu_line.upper()))
kwargs['CPPDEFINES'] = cppdefs

#
# CPPPATH
#
subdirs = ['inc', 'tpl', 'CMSIS/Device/ST/STM32F10x/Include' ]
cpppath = env.get('CPPPATH',[]) + kwargs.get('CPPPATH',[])
cpppath += [ '%s/%s' % (stdperiph_dir, s) for s in subdirs ]
cpppath += ['#CMSIS/Include']
kwargs['CPPPATH'] = cpppath

#
# CFLAGS
#
cflags = env.get('CFLAGS',[]) + kwargs.get('CFLAGS', [])
cflags += mcu_flags
kwargs['CFLAGS'] = cflags

#
# CXXFLAGS
#
cxxflags = env.get('CFLAGS',[]) + kwargs.get('CFLAGS', [])
cxxflags += mcu_flags
kwargs['CXXFLAGS'] = cxxflags

#
# LINKFLAGS
#
linkflags = env.get('CFLAGS',[]) + kwargs.get('CFLAGS', [])
linkflags += mcu_flags
kwargs['LINKFLAGS'] = linkflags

#
# SOURCES
#
subdirs = ['src', 'tpl']
sources = [ env.Glob('%s/%s/*.c' % (stdperiph_dir, s)) for s in subdirs ]
sources = Flatten(sources)

#
# LIBRARY NAME
#
libname = "stdperiph_%s_%s" % (mcu_family.lower(), mcu_line.lower())

#
# BUILD THE LIBRARY
#
target  = env.Library(libname, sources, **kwargs)

#
# RETURN WHAT WAS BUILT
#
Return('target')

# Local Variables:
# # tab-width:4
# # indent-tabs-mode:nil
# # End:
# vim: set syntax=scons expandtab tabstop=4 shiftwidth=4:
