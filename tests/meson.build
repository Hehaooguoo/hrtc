
icetestsrc = ['icetest.cpp','../ice/agent.cpp']

icetest = executable('icetest', icetestsrc,
  install : true,include_directories : hrct_incs)

test('command line test', icetest, args : ['-a', '-b'])
test('command line test', icetest, args : ['-ab'])
test('command line test', icetest, args : ['-c', 'foo'])
test('command line test', icetest, args : ['-cfoo'])
test('command line test', icetest, args : ['arg1'])
test('command line test', icetest, args : ['-a','--' ,'-b'])
test('command line test', icetest, args : [' -a','-'])
