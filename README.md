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
    results = self.cmd.get_epilog()
    self.assertIn('hook epilog', results)
    
  def test_before(self):
    self.assertFalse(self.hook._before_called)
    parser = self.cmd.get_parser('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    self.cmd.run(results)
    self.assertTrue(self.hook._before_called)
    self.assertEqual(results.added_by_hook, 'othervalue')
    self.assertTrue(results.added_by_before)
    
  def test_after(self):
    self.assertFalse(self.hook._after_called)
    parser = self.cmd.get_parser('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    result = self.cmd.run(results)
    self.assertTrue(self.hook._after_called)
    self.assertEqual(result, 0)
    
class TestListerHooks(base.TestBase):
  def setUp(self):
    super(TestListerHooks, self).setUp()
    self.app = make_app()
    self.cmd = TestListerCommand(self.app, None, cmd_name='test')
    self.hook = TestHook(self.cmd)
    self.mgr = extension.ExtensionManager.make_test_instance(
      [extension.Extension(
        'parser-hook',
        None,
        None,
        self.hook)],
    )
    self.cmd._hooks = self.mgr
    
  def test_get_parser(self):
    parser = self.cmd.get_parser('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    self.assertEqual(results.added_by_hook, 'value')
    
  def test_get_epilog(self):
    results = self.cmd.get_epilog('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    self.assertEqual(results.added_by_hook, 'value')
    
  def test_before(self):
    self.assertFalse(self.hook._before_called)
    parser = self.cmd.get_parser('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    self.cmd.run(results)
    self.assertTrue(self.hook._before_called)
    
  def test_after(self):
    self.assertFalse(self.hook._after_called)
    parser = self.cmd.get_parser('test')
    results = parser.parser_args(['--added-by-hook', 'value'])
    self.cmd.run(results)
    self.assertTrue(self.hook._after_called)
  
class TEstListerChangeHooks(base.TestBase):
  def setUp(self):
    super().setUp()
    self.app = make_app()
    self.cmd = TestListerCommand()
    self.hook = TestListerChangeHook()
    self.mgr = extension.ExtensionManager.make_test_instance(
      [extension.Extension(
        'parser-hook',
        None,
        None,
        self.hook)],
    )
    self.cmd_hooks = self.mgr

  def test_get_parser(self):
    parser = self.cmd.get_parser('test')
    results = parser.parse_args(['--added-by-hook', 'value'])
    self.assertEqual(results.added_by_hook, 'value')
    
  def test_get_epilog(self):
    results = self.cmd.get_epilog()
    self.assertIn('hook epilog', results)
    
  def test_before(self):
    self.assertFalse(self.hook._before_called)
    parser = self.cmd.get_parser('test')
    results = parser.parse_args(['--added-by-hook', 'value'])
    self.cmd.run(results)
    self.assertTrue(self.hook._before_called)
    self.assertEqual(results.added_by_hook, 'othervalue')
    self.assertTrue(results.added_by_before)

  def test_after(self):
    self.assertFalse(self.hook._after_called)
    parser = self.cmd.get_parser('test')
    results = parser.parse_args(['--added-by-hook', 'value'])
    result = self.cmd.run(results)
    self.assertTrue(self.hook._after_called)
    self.assertEqual(result, 0)
```

```
```

```
```


