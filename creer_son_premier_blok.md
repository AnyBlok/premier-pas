# Créer son premier Blok

                                                                                
1) Créer un paquet python                                                       
                                                                                
lire le tutaux sur les paquet python pour le pypi                               
http://sametmax.com/creer-un-setup-py-et-mettre-sa-bibliotheque-python-en-ligne-sur-pypi/
                                                                                
Structure du package                                                            
                                                                                
anyblok_todolist                                                                
.. anyblok_todolist                                                             
.. .. __init__.py                                                               
.. .. todolist                                                                  
.. .. .. __init__.py                                                            
.. setup.py                                                                     
                                                                                
1.1) Fichier setup                                                              
                                                                                
::                                                                              
                                                                                
    from setuptools import setup, find_packages                                 
                                                                                
    requires = [                                                                
        'anyblok',                                                              
    ]                                                                           
                                                                                
    setup(                                                                      
        name="anyblok_todolist",                                                
        version='0.1.0',                                                        
        author="Jean-Sébastien Suzanne",                                        
        author_email="jssuzanne@anybox.fr",                                     
        description="Todolist for AnyBlok",                                     
        license="MPL2",                                                         
        long_description="Tiny todolist for AnyBlok",                           
        packages=find_packages(),                                               
        zip_safe=False,                                                         
        include_package_data=True,                                              
        install_requires=requires,                                                 
        tests_require=requires + ['nose'],                                      
        classifiers=[                                                           
            'Intended Audience :: Developers',                                  
            'Programming Language :: Python :: 3.3',                            
            'Programming Language :: Python :: 3.4',                            
            'Programming Language :: Python :: 3.5',                            
            'Topic :: Software Development :: Libraries :: Python Modules',     
        ],                                                                      
        entry_points={                                                          
            'bloks': [                                                          
                'todolist=anyblok_todolist.todolist:BlokTodoList',              
            ],                                                                  
        },                                                                      
        extras_require={},                                                      
    )                        
                                                                                   
1.2) Fichier de description du blok ``anyblok_todolist/anyblok_todolist/todolist/__init__.py``
                                                                                
::                                                                              
                                                                                
    from anyblok.blok import Blok                                               
                                                                                
                                                                                
    class BlokTodoList(Blok):                                                   
        version = '0.1.0'                                                       
                                                                                
                                                                                
2) Installer le nouveau package.                                                
                                                                                
cd anyblok_todolist                                                             
../sandbox/bin/python setup.py develop                                          
cd ..                                                                           
                                                                                
3) Vérifier que le nouveau blok existe.                                         
                                                                                
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.System.Blok.query().all()                                  
==> Out[1]:                                                                     
==> [anyblok-core (installed),                                                  
==>  anyblok-io (uninstalled),                                                  
==>  anyblok-io-csv (uninstalled),                                              
==>  anyblok-io-xml (uninstalled),                                              
==>  model_authz (uninstalled),                                                 
==>  todolist (uninstalled)]                                                    
                                                                                
4) Installer le blok todolist                                                   
                                                                                
sandbox/bin/anyblok_updatedb --db-name todolist --db-driver-name postgresql --install-bloks todolist
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.System.Blok.query().all()                                  
==> Out[1]:                                                                     
==> [anyblok-core (installed),                                                  
==>  anyblok-io (uninstalled),                                                  
==>  anyblok-io-csv (uninstalled),                                              
==>  anyblok-io-xml (uninstalled),                                              
==>  model_authz (uninstalled),                                                 
==>  todolist (installed)]       

                                                                                
5) Lister les ``Model`` existant.                                               
                                                                                
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.System.Model.query().all().name                            
==> Out[1]:                                                                     
==> ['Model.System',                                                            
==>  'Model.System.Column',                                                     
==>  'Model.Authorization',                                                     
==>  'Model.Documentation.Blok',                                                
==>  'Model.System.Parameter',                                                  
==>  'Model.System.Cron',                                                       
==>  'Model.Documentation.Model',                                               
==>  'Model.System.Cache',                                                      
==>  'Model.Documentation.Model.Field',                                         
==>  'Model.System.Model',                                                      
==>  'Model.System.Sequence',                                                   
==>  'Model.System.Cron.Worker',                                                
==>  'Model.System.Blok',                                                       
==>  'Model.System.Cron.Job',                                                   
==>  'Model.System.Field',                                                      
==>  'Model.Documentation.Model.Attribute',                                     
==>  'Model.System.RelationShip',                                               
==>  'Model.Documentation']         

                                                                                
6) Ajout du model ``Todo``                                                      
                                                                                
6.1) Créer le module ``todo.py`` dans le blok ``todolist``                      
                                                                                
::                                                                              
                                                                                
    from anyblok import Declarations                                                                                                               
    from anyblok.column import Integer, String, Text, DateTime, Selection                                                                          
    from datetime import datetime                                                                                                                  
                                                                                
                                                                                
    @Declarations.register(Declarations.Model)                                                                                                     
    class Todo:                                                                 
                                                                                
        id = Integer(primary_key=True)                                                                                                             
        title = String(nullable=False)                                                                                                             
        description = Text()                                                                                                                       
        creation_date = DateTime(default=datetime.now, nullable=False)                                                                             
        deadline_date = DateTime()                                                                                                                 
        ending_date = DateTime()                                                                                                                   
        cancelled_date = DateTime()                                                 
        step = Selection(selections=[('todo', 'Todo'), ('done', 'Done'),            
                                     ('canceled', 'Canceled')],                     
                         nullable=False, default='todo')                            
        complexity = Selection(selections=[('veasy', 'Very easy'),                  
                                           ('easy', 'Easy'),                        
                                           ('normal', 'Normal'),                    
                                           ('dificulte', 'Dificulte'),              
                                           ('vdificulte', 'Very dificulte')],       
                               nullable=False, default='normal')                    
        priority = Selection(selections=[('3', 'Low'),                              
                                         ('2', 'Normal'),                           
                                         ('1', 'Hight')],                           
                             nullable=False, default='2')  
                             
                                                                                
6.2) Declarer le fichier todo dans le blok                                      
                                                                                
::                                                                              
                                                                                
    ...                                                                         
                                                                                
    class BlokTodoList(Blok):                                                                                                                      
                                                                                
        ...                                                                     
                                                                                                                                                       
        @classmethod                                                                                                                               
        def import_declaration_module(cls):                                                                                                        
            from . import todo  # noqa                                                                                                             
                                                                                
        @classmethod                                                                                                                               
        def reload_declaration_module(cls, reload):                                                                                                
            from . import todo                                                                                                                     
            reload(todo)                                                        
                                                                                
6.3) Lister les models et fields                                                
                                                                                
                                                                                
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.System.Model.query().all().name                            
==> Out[1]:                                                                     
==> ['Model.System',                                                            
==>  'Model.System.Column',                                                     
==>  'Model.Authorization',                                                     
==>  'Model.Documentation.Blok',                                                
==>  'Model.System.Parameter',                                                  
==>  'Model.System.Cron',                                                       
==>  'Model.Documentation.Model',                                               
==>  'Model.System.Cache',                                                      
==>  'Model.Documentation.Model.Field',                                         
==>  'Model.System.Model',                                                      
==>  'Model.System.Sequence',                                                   
==>  'Model.System.Cron.Worker',                                                
==>  'Model.System.Blok',                                                       
==>  'Model.System.Cron.Job',                                                   
==>  'Model.System.Field',                                                      
==>  'Model.Documentation.Model.Attribute',                                     
==>  'Model.System.RelationShip',                                               
==>  'Model.Documentation',                                                     
==>  'Model.Todo']                                                              
==> In [2]: registry.System.Field.query().filter_by(model='Model.Todo').all().name
                                                                                
==> Out[2]:                                                                     
==> ['cancelled_date',                                                          
==>  'title',                                                                   
==>  'description',                                                             
==>  'complexity',                                                              
==>  'step',                                                                    
==>  'priority',                                                                
==>  'creation_date',                                                           
==>  'id',                                                                      
==>  'ending_date',                                                             
==>  'deadline_date'] 

6.4) Lister le schéma                                                           
                                                                                
psql todolist                                                                   
==> \d todo                                                                     
                        Table "public.todo"                                     
     Column     |           Type           |                     Modifiers                     
----------------+--------------------------+---------------------------------------------------
 cancelled_date | timestamp with time zone |                                    
 title          | character varying(64)    | not null                           
 description    | text                     |                                    
 complexity     | character varying(64)    | not null                           
 step           | character varying(64)    | not null                           
 priority       | character varying(64)    | not null                           
 creation_date  | timestamp with time zone | not null                           
 id             | integer                  | not null default nextval('todo_id_seq'::regclass)
 ending_date    | timestamp with time zone |                                    
 deadline_date  | timestamp with time zone |                                    
                                                                                
Indexes:                                                                        
 "anyblok_pk_todo" PRIMARY KEY, btree (id)                                      
                                                                                
6.5) Enregister des données                                                     
                                                                                   
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.Todo.insert(title="My first todo entry in the todolist")   
==> Out[1]: <anyblok.model.todo at 0x111353d30>                                 
                                                                                
==> In [2]: registry.Todo.query().all()                                         
==> Out[2]: [<anyblok.model.todo at 0x111353d30>]                               
                                                                                
==> In [3]: registry.commit()           

                                                                               
7) Improve repr and str method to have a better visibility                      
                                                                                
::                                                                              
                                                                                
    ...                                                                         
                                                                                
    @Declarations.register(Declarations.Model)                                       
    class Todo:                                                                    
                                                                                
        ...                                                                     
                                                                                
        def __str__(self):                                                           
            return ('{self.step.label}: {self.title} '                               
                    '[{self.priority.label} / {self.complexity.label}]').format(     
                self=self)                                                           
                                                                                                                 
        def __repr__(self):                                                          
            msg = ('<TodoList: title={self.title}, '                                 
                   'step={self.step.label}, '                                        
                   'creation date={self.creation_date}')                             
                                                                                
            if self.deadline_date:                                                   
                msg += ', deadline={self.deadline_date}'                             
                                                                                
            if self.step == 'canceled':                                              
                msg += ', canceled date={self.cancelled_date}'                       
            elif self.step == 'done':                                                
                msg += ', canceled date={self.ending_date}'                          
            else:                                                                    
                msg += (', priority={self.priority.label}'                           
                        ', complexity={self.complexity.label}')                      
                                                                                
            msg += ' >'                                                              
            return msg.format(self=self)                                        
                                                                                
                                                                                
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: registry.Todo.query().all()                                         
==> Out[1]: [<TodoList: title=My first todo entry in the todolist, step=Todo, creation date=2016-05-11 10:50:50.293404+02:00, priority=Normal, complexity=Normal >]
==>                                                                             
==> In [2]: str(registry.Todo.query().first())                                  
==> Out[2]: 'Todo: My first todo entry in the todolist [Normal / Normal]'       
                                                                                
8) Add workflow                                                                 
                                                                                
Add exceptions.py file in the blok                                              
                                                                                
::                                                                              
                                                                                
    class TodoException(Exception):                                             
        pass                                                                    
                                                                                
                                                                                
    class TodoExceptionDone(TodoException):                                     
        pass                                                                    
                                                                                
                                                                                
    class TodoExceptionCanceled(TodoException):                                    
        pass                                                                    
                                                                                
                                                                                
::                                                                              
                                                                                
    ...                                                                         
                                                                                
    @Declarations.register(Declarations.Model)                                       
    class Todo:                                                                    
                                                                                
        ...                                                                     
                                                                                
                                                                                
        def do(self):                                                               
            if self.step != 'todo':                                                 
                raise TodoExceptionDone("You can change step %r to 'Done'" %        
                                        self.step.label)                            
            self.step = 'done'                                                      
            self.ending_date = datetime.now()                                       
                                                                                
        def cancel(self):                                                           
            if self.step != 'todo':                                                 
                raise TodoExceptionCanceled("You can change step %r to 'Canceled'"  
                                            % self.step.label)                      
            self.step = 'canceled'                                                    
            self.ending_date = datetime.now()                                   
                                                           
                                                                                  
sandbox/bin/anyblok_interpreter --db-name todolist --db-driver-name postgresql     
==> In [1]: todo = registry.Todo.query().first()                                
                                                                                
==> In [2]: todo                                                                
==> Out[2]: <TodoList: title=My first todo entry in the todolist, step=Todo, creation date=2016-05-11 10:50:50.293404+02:00, priority=Normal, complexity=Normal >
                                                                                
==> In [3]: todo.do()                                                           
                                                                                
==> In [4]: todo                                                                
==> Out[4]: <TodoList: title=My first todo entry in the todolist, step=Done, creation date=2016-05-11 10:50:50.293404+02:00, canceled date=2016-05-11 10:22:26.515175+01:00 >
                                                                                
==> In [5]: todo.cancel()                                                       
    --------------------------------------------------------------------------- 
    TodoExceptionCanceled                     Traceback (most recent call last) 
    /Users/jssuzanne/anybox/tuto_anyblok/sandbox/lib/python3.5/site-packages/anyblok/scripts.py in <module>()
    ----> 1 todo.cancel()                                                       
                                                                                
    /Users/jssuzanne/anybox/tuto_anyblok/anyblok_todolist/anyblok_todolist/todolist/todo.py in cancel(self)
         63         if self.step != 'todo':                                     
         64             raise TodoExceptionCanceled("You can change step %r to 'Canceled'"
    ---> 65                                         % self.step.label)          
         66         self.step = 'cancel'                                        
         67         self.ending_date = datetime.now()                           
                                                                                
    TodoExceptionCanceled: You can change step 'Done' to 'Canceled'             
                                                                     
                                                                                 
9) Add unit test                                                                
                                                                                
in the blok add the files:                                                      
                                                                                
todolist                                                                        
.. __init__.py                                                                  
.. exceptions.py                                                                
.. todo.py                                                                      
.. tests                                                                        
.. .. __init__.py                                                               
.. .. test_todo.py                                                              
                                                                                
in the file test_dodo.py                                                        
                                                                                
::                                                                              
                                                                                
    from anyblok.tests.testcase import BlokTestCase                             
    from ..exceptions import TodoException                                      
                                                                                
                                                                                
    class TestTodo(BlokTestCase):                                                                                                                  
                                                                                
        def test_do(self):                                                      
            todo = self.registry.Todo.insert(title='Test')                      
            self.assertEqual(todo.step, 'todo')                                 
            todo.do()                                                           
            self.assertEqual(todo.step, 'done')                                 
                                                                                
        def test_do_with_bad_step(self):                                        
            todo = self.registry.Todo.insert(title='Test')                      
            self.assertEqual(todo.step, 'todo')                                 
            todo.step = 'canceled'                                              
            with self.assertRaises(TodoException):                              
                todo.do()                                                       
                                                                                
         def test_cancel(self):                                                 
             todo = self.registry.Todo.insert(title='Test')                     
             self.assertEqual(todo.step, 'todo')                                
             todo.cancel()                                                      
             self.assertEqual(todo.step, 'canceled')                            
                                                                                
         def test_cancel_with_bad_step(self):                                   
             todo = self.registry.Todo.insert(title='Test')                     
             self.assertEqual(todo.step, 'todo')                                
             todo.step = 'done'                                                 
             with self.assertRaises(TodoException):                             
                 todo.cancel()                                                  
                                                                                
Run the tests                                                                   
                                                                                
jssuzanne:tuto_anyblok jssuzanne$ sandbox/bin/anyblok_nose --db-name todolist --db-driver-name postgresql --selected-bloks todolist
Load config file '/Library/Application Support/AnyBlok/conf.cfg'                
Load config file '/Users/jssuzanne/Library/Application Support/AnyBlok/conf.cfg'
....                                                                            
----------------------------------------------------------------------          
Ran 4 tests in 0.880s                                                           
                                                                                
OK                                                                              
                                                                                
Congrate you have your first blok                                               
                                   