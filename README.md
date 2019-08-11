### cliff
---
https://github.com/openstack/cliff

https://docs.openstack.org/cliff/latest/

```py
// cliff/tests/test_command_hooks.py
from cliff import app as application
from cliff import command
from cliff import commandmanager
from cliff import hooks
from cliff import lister
from cliff.tests import base

import mock
from stevedore import extension

def make_app(**kwargs):
  cmd_mgr = commandmanager.CommandManager('cliff.tests')
  
  cmd = mock.MagicMock(spec=command.Command)
  command_inst = mock.MagicMock(spec=command.Command)
  command_inst.run.return_value = 0
  cmd.return_value = command_inst
  cmd_mgr.add_command('mock', cmd)
  
  err_command = mock.MagicMock(name='err_command', spec=command.Command)
  err_command_inst = mock.Mock(spec=command.Command)
  err_command_inst.run = mock.Mock(
    side_effect=RuntimeError('test exception')
  )
  err_command.return_value = err_command_inst
  cmd_mgr.add_command('error', err_command)
  
  app = application.App('testing command hooks',
    '1',
    cmd_mgr,
    stderr=mock.Mock(),
    **kwargs
    )
  return app
  
class TestCommand(command.Command):
  """
  """
  
  def get_parser(self, prog_name):
    parser = super(TestCommand, self).get_parser(prog_name)
    return parser
    
  def take_action(self, parserd_args):
    return 42
    
class TestShowCommand(show.ShowOne):
  """
  """
  def take_action(self, parsed_args):
    return (('Name',), ('value',))

class TestListerCommand(lister.Lister):
  """
  """
  def take_action(self, parsed_args):
    return (('Name',), [('value',)])

class TestHook(hooks.CommandHook):
  _before_called = False
  _after_called = False
  
  def get_parser(self, parser):
    parser.add_argument('--added-by-hook')
    return parser
    
  def get_epilog(self):
    return 'hook epilog'
    
  def before(self, parsed_args):
    self._before_called = True
    parsed_args.added_by_hook = 'othervalue'
    parsed_args.added_by_before = True
    return parsed_args
    
  def after(self, parsed_args, return_code):
    self._after_called = True
    return 24
    
class TestDisplayChangeHook(hooks.CommandHook):
  _before_called = False
  _after_called = False
  
  def get_parser(self, parser):
  



```

```
```

```
```


