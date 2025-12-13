# 控制流图

> [`www.fuzzingbook.org/html/ControlFlow.html`](http://www.fuzzingbook.org/html/ControlFlow.html)

这个笔记本中的代码有助于获取 Python 函数的控制流图。

**先决条件**

+   这个笔记本需要对 Python 的高级概念有所了解，特别是

    +   类

## 控制流图

`PyCFG`类允许用户获取控制流图。

```py
from ControlFlow import gen_cfg, to_graph
cfg = gen_cfg(inspect.getsource(my_function))
to_graph(cfg) 
```

```py
import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) 
```

```py
from [bookutils](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils) import print_content 
```

```py
import [ast](https://docs.python.org/3/library/ast.html)
import [re](https://docs.python.org/3/library/re.html) 
```

```py
from [graphviz](https://graphviz.readthedocs.io/) import Source, Digraph 
```

### 注册

```py
REGISTRY_IDX = 0 
```

```py
REGISTRY = {} 
```

```py
def get_registry_idx():
    global REGISTRY_IDX
    v = REGISTRY_IDX
    REGISTRY_IDX += 1
    return v 
```

```py
def reset_registry():
    global REGISTRY_IDX
    global REGISTRY
    REGISTRY_IDX = 0
    REGISTRY = {} 
```

```py
def register_node(node):
    node.rid = get_registry_idx()
    REGISTRY[node.rid] = node 
```

```py
def get_registry():
    return dict(REGISTRY) 
```

### CFGNode

我们从表示控制流图每个节点的`CFGNode`开始。\todo{增强和注解赋值（`a += 1`），（`a:int = 1`）}。

```py
class CFGNode(dict):
    def __init__(self, parents=[], ast=None):
        assert type(parents) is list
        register_node(self)
        self.parents = parents
        self.ast_node = ast
        self.update_children(parents)  # requires self.rid
        self.children = []
        self.calls = []

    def i(self):
        return str(self.rid)

    def update_children(self, parents):
        for p in parents:
            p.add_child(self)

    def add_child(self, c):
        if c not in self.children:
            self.children.append(c)

    def lineno(self):
        return self.ast_node.lineno if hasattr(self.ast_node, 'lineno') else 0

    def __str__(self):
        return "id:%d line[%d] parents: %s : %s" % (
            self.rid, self.lineno(), str([p.rid for p in self.parents]),
            self.source())

    def __repr__(self):
        return str(self)

    def __eq__(self, other):
        return self.rid == other.rid

    def __neq__(self, other):
        return self.rid != other.rid

    def set_parents(self, p):
        self.parents = p

    def add_parent(self, p):
        if p not in self.parents:
            self.parents.append(p)

    def add_parents(self, ps):
        for p in ps:
            self.add_parent(p)

    def add_calls(self, func):
        self.calls.append(func)

    def source(self):
        return ast.unparse(self.ast_node).strip()

    def to_json(self):
        return {
            'id': self.rid,
            'parents': [p.rid for p in self.parents],
            'children': [c.rid for c in self.children],
            'calls': self.calls,
            'at': self.lineno(),
            'ast': self.source()
        } 
```

### PyCFG

接下来是负责解析和持有图的`PyCFG`类。

```py
class PyCFG:
    def __init__(self):
        self.founder = CFGNode(
            parents=[], ast=ast.parse('start').body[0])  # sentinel
        self.founder.ast_node.lineno = 0
        self.functions = {}
        self.functions_node = {} 
```

```py
class PyCFG(PyCFG):
    def parse(self, src):
        return ast.parse(src) 
```

```py
class PyCFG(PyCFG):
    def walk(self, node, myparents):
        fname = "on_%s" % node.__class__.__name__.lower()
        if hasattr(self, fname):
            fn = getattr(self, fname)
            v = fn(node, myparents)
            return v
        else:
            return myparents 
```

```py
class PyCFG(PyCFG):
    def on_module(self, node, myparents):
  """
 Module(stmt* body)
 """
        # each time a statement is executed unconditionally, make a link from
        # the result to next statement
        p = myparents
        for n in node.body:
            p = self.walk(n, p)
        return p 
```

```py
class PyCFG(PyCFG):
    def on_augassign(self, node, myparents):
  """
 AugAssign(expr target, operator op, expr value)
 """
        p = [CFGNode(parents=myparents, ast=node)]
        p = self.walk(node.value, p)

        return p 
```

```py
class PyCFG(PyCFG):
    def on_annassign(self, node, myparents):
  """
 AnnAssign(expr target, expr annotation, expr? value, int simple)
 """
        p = [CFGNode(parents=myparents, ast=node)]
        p = self.walk(node.value, p)

        return p 
```

```py
class PyCFG(PyCFG):
    def on_assign(self, node, myparents):
  """
 Assign(expr* targets, expr value)
 """
        if len(node.targets) > 1:
            raise NotImplemented('Parallel assignments')

        p = [CFGNode(parents=myparents, ast=node)]
        p = self.walk(node.value, p)

        return p 
```

```py
class PyCFG(PyCFG):
    def on_pass(self, node, myparents):
        return [CFGNode(parents=myparents, ast=node)] 
```

```py
class PyCFG(PyCFG):
    def on_break(self, node, myparents):
        parent = myparents[0]
        while not hasattr(parent, 'exit_nodes'):
            # we have ordered parents
            parent = parent.parents[0]

        assert hasattr(parent, 'exit_nodes')
        p = CFGNode(parents=myparents, ast=node)

        # make the break one of the parents of label node.
        parent.exit_nodes.append(p)

        # break doesn't have immediate children
        return [] 
```

```py
class PyCFG(PyCFG):
    def on_continue(self, node, myparents):
        parent = myparents[0]
        while not hasattr(parent, 'exit_nodes'):
            # we have ordered parents
            parent = parent.parents[0]
        assert hasattr(parent, 'exit_nodes')
        p = CFGNode(parents=myparents, ast=node)

        # make continue one of the parents of the original test node.
        parent.add_parent(p)

        # return the parent because a continue is not the parent
        # for the just next node
        return [] 
```

```py
class PyCFG(PyCFG):
    def on_for(self, node, myparents):
        # node.target in node.iter: node.body
        # The For loop in python (no else) can be translated
        # as follows:
        # 
        # for a in iterator:
        #      mystatements
        #
        # __iv = iter(iterator)
        # while __iv.__length_hint() > 0:
        #     a = next(__iv)
        #     mystatements

        init_node = CFGNode(parents=myparents,
            ast=ast.parse('__iv = iter(%s)' % ast.unparse(node.iter).strip()).body[0])
        ast.copy_location(init_node.ast_node, node.iter)

        _test_node = CFGNode(
            parents=[init_node],
            ast=ast.parse('_for: __iv.__length__hint__() > 0').body[0])
        ast.copy_location(_test_node.ast_node, node)

        # we attach the label node here so that break can find it.
        _test_node.exit_nodes = []
        test_node = self.walk(node.iter, [_test_node])

        extract_node = CFGNode(parents=test_node,
            ast=ast.parse('%s = next(__iv)' % ast.unparse(node.target).strip()).body[0])
        ast.copy_location(extract_node.ast_node, node.iter)

        # now we evaluate the body, one at a time.
        p1 = [extract_node]
        for n in node.body:
            p1 = self.walk(n, p1)

        # the test node is looped back at the end of processing.
        _test_node.add_parents(p1)

        return _test_node.exit_nodes + test_node 
```

```py
class PyCFG(PyCFG):
    def on_while(self, node, myparents):
        # For a while, the earliest parent is the node.test
        _test_node = CFGNode(
            parents=myparents,
            ast=ast.parse(
                '_while: %s' % ast.unparse(node.test).strip()).body[0])
        ast.copy_location(_test_node.ast_node, node.test)
        _test_node.exit_nodes = []
        test_node = self.walk(node.test, [_test_node])

        # we attach the label node here so that break can find it.

        # now we evaluate the body, one at a time.
        assert len(test_node) == 1
        p1 = test_node
        for n in node.body:
            p1 = self.walk(n, p1)

        # the test node is looped back at the end of processing.
        _test_node.add_parents(p1)

        # link label node back to the condition.
        return _test_node.exit_nodes + test_node 
```

```py
class PyCFG(PyCFG):
    def on_if(self, node, myparents):
        _test_node = CFGNode(
            parents=myparents,
            ast=ast.parse(
                '_if: %s' % ast.unparse(node.test).strip()).body[0])
        ast.copy_location(_test_node.ast_node, node.test)
        test_node = self.walk(node.test, [ _test_node])
        assert len(test_node) == 1
        g1 = test_node
        for n in node.body:
            g1 = self.walk(n, g1)
        g2 = test_node
        for n in node.orelse:
            g2 = self.walk(n, g2)
        return g1 + g2 
```

```py
class PyCFG(PyCFG):
    def on_binop(self, node, myparents):
        left = self.walk(node.left, myparents)
        right = self.walk(node.right, left)
        return right 
```

```py
class PyCFG(PyCFG):
    def on_compare(self, node, myparents):
        left = self.walk(node.left, myparents)
        right = self.walk(node.comparators[0], left)
        return right 
```

```py
class PyCFG(PyCFG):
    def on_unaryop(self, node, myparents):
        return self.walk(node.operand, myparents) 
```

```py
class PyCFG(PyCFG):
    def on_call(self, node, myparents):
        def get_func(node):
            if type(node.func) is ast.Name:
                mid = node.func.id
            elif type(node.func) is ast.Attribute:
                mid = node.func.attr
            elif type(node.func) is ast.Call:
                mid = get_func(node.func)
            else:
                raise Exception(str(type(node.func)))
            return mid
            #mid = node.func.value.id

        p = myparents
        for a in node.args:
            p = self.walk(a, p)
        mid = get_func(node)
        myparents[0].add_calls(mid)

        # these need to be unlinked later if our module actually defines these
        # functions. Otherwise we may leave them around.
        # during a call, the direct child is not the next
        # statement in text.
        for c in p:
            c.calllink = 0
        return p 
```

```py
class PyCFG(PyCFG):
    def on_expr(self, node, myparents):
        p = [CFGNode(parents=myparents, ast=node)]
        return self.walk(node.value, p) 
```

```py
class PyCFG(PyCFG):
    def on_return(self, node, myparents):
        if type(myparents) is tuple:
            parent = myparents[0][0]
        else:
            parent = myparents[0]

        val_node = self.walk(node.value, myparents)
        # on return look back to the function definition.
        while not hasattr(parent, 'return_nodes'):
            parent = parent.parents[0]
        assert hasattr(parent, 'return_nodes')

        p = CFGNode(parents=val_node, ast=node)

        # make the break one of the parents of label node.
        parent.return_nodes.append(p)

        # return doesnt have immediate children
        return [] 
```

```py
class PyCFG(PyCFG):
    def on_functiondef(self, node, myparents):
        # a function definition does not actually continue the thread of
        # control flow
        # name, args, body, decorator_list, returns
        fname = node.name
        args = node.args
        returns = node.returns

        enter_node = CFGNode(
            parents=[],
            ast=ast.parse('enter: %s(%s)' % (node.name, ', '.join(
                [a.arg for a in node.args.args]))).body[0])  # sentinel
        enter_node.calleelink = True
        ast.copy_location(enter_node.ast_node, node)
        exit_node = CFGNode(
            parents=[],
            ast=ast.parse('exit: %s(%s)' % (node.name, ', '.join(
                [a.arg for a in node.args.args]))).body[0])  # sentinel
        exit_node.fn_exit_node = True
        ast.copy_location(exit_node.ast_node, node)
        enter_node.return_nodes = []  # sentinel

        p = [enter_node]
        for n in node.body:
            p = self.walk(n, p)

        for n in p:
            if n not in enter_node.return_nodes:
                enter_node.return_nodes.append(n)

        for n in enter_node.return_nodes:
            exit_node.add_parent(n)

        self.functions[fname] = [enter_node, exit_node]
        self.functions_node[enter_node.lineno()] = fname

        return myparents 
```

```py
class PyCFG(PyCFG):
    def get_defining_function(self, node):
        if node.lineno() in self.functions_node:
            return self.functions_node[node.lineno()]
        if not node.parents:
            self.functions_node[node.lineno()] = ''
            return ''
        val = self.get_defining_function(node.parents[0])
        self.functions_node[node.lineno()] = val
        return val 
```

```py
class PyCFG(PyCFG):
    def link_functions(self):
        for nid, node in REGISTRY.items():
            if node.calls:
                for calls in node.calls:
                    if calls in self.functions:
                        enter, exit = self.functions[calls]
                        enter.add_parent(node)
                        if node.children:
                            # # until we link the functions up, the node
                            # # should only have succeeding node in text as
                            # # children.
                            # assert(len(node.children) == 1)
                            # passn = node.children[0]
                            # # We require a single pass statement after every
                            # # call (which means no complex expressions)
                            # assert(type(passn.ast_node) == ast.Pass)

                            # # unlink the call statement
                            assert node.calllink > -1
                            node.calllink += 1
                            for i in node.children:
                                i.add_parent(exit)
                            # passn.set_parents([exit])
                            # ast.copy_location(exit.ast_node, passn.ast_node)

                            # #for c in passn.children: c.add_parent(exit)
                            # #passn.ast_node = exit.ast_node 
```

```py
class PyCFG(PyCFG):
    def update_functions(self):
        for nid, node in REGISTRY.items():
            _n = self.get_defining_function(node) 
```

```py
class PyCFG(PyCFG):
    def update_children(self):
        for nid, node in REGISTRY.items():
            for p in node.parents:
                p.add_child(node) 
```

```py
class PyCFG(PyCFG):
    def gen_cfg(self, src):
  """
 >>> i = PyCFG()
 >>> i.walk("100")
 5
 """
        node = self.parse(src)
        nodes = self.walk(node, [self.founder])
        self.last_node = CFGNode(parents=nodes, ast=ast.parse('stop').body[0])
        ast.copy_location(self.last_node.ast_node, self.founder.ast_node)
        self.update_children()
        self.update_functions()
        self.link_functions() 
```

### 支持函数

```py
def compute_dominator(cfg, start=0, key='parents'):
    dominator = {}
    dominator[start] = {start}
    all_nodes = set(cfg.keys())
    rem_nodes = all_nodes - {start}
    for n in rem_nodes:
        dominator[n] = all_nodes

    c = True
    while c:
        c = False
        for n in rem_nodes:
            pred_n = cfg[n][key]
            doms = [dominator[p] for p in pred_n]
            i = set.intersection(*doms) if doms else set()
            v = {n} | i
            if dominator[n] != v:
                c = True
            dominator[n] = v
    return dominator 
```

```py
def compute_flow(pythonfile):
    cfg, first, last = get_cfg(pythonfile)
    return cfg, compute_dominator(
        cfg, start=first), compute_dominator(
            cfg, start=last, key='children') 
```

```py
def gen_cfg(fnsrc, remove_start_stop=True):
    reset_registry()
    cfg = PyCFG()
    cfg.gen_cfg(fnsrc)
    cache = dict(REGISTRY)
    if remove_start_stop:
        return {
            k: cache[k]
            for k in cache if cache[k].source() not in {'start', 'stop'}
        }
    else:
        return cache 
```

```py
def get_cfg(src):
    reset_registry()
    cfg = PyCFG()
    cfg.gen_cfg(src)
    cache = dict(REGISTRY)
    g = {}
    for k, v in cache.items():
        j = v.to_json()
        at = j['at']
        parents_at = [cache[p].to_json()['at'] for p in j['parents']]
        children_at = [cache[c].to_json()['at'] for c in j['children']]
        if at not in g:
            g[at] = {'parents': set(), 'children': set()}
        # remove dummy nodes
        ps = set([p for p in parents_at if p != at])
        cs = set([c for c in children_at if c != at])
        g[at]['parents'] |= ps
        g[at]['children'] |= cs
        if v.calls:
            g[at]['calls'] = v.calls
        g[at]['function'] = cfg.functions_node[v.lineno()]
    return (g, cfg.founder.ast_node.lineno, cfg.last_node.ast_node.lineno) 
```

```py
def to_graph(cache, arcs=[]):
    graph = Digraph(comment='Control Flow Graph')
    colors = {0: 'blue', 1: 'red'}
    kind = {0: 'T', 1: 'F'}
    cov_lines = set(i for i, j in arcs)
    for nid, cnode in cache.items():
        lineno = cnode.lineno()
        shape, peripheries = 'oval', '1'
        if isinstance(cnode.ast_node, ast.AnnAssign):
            if cnode.ast_node.target.id in {'_if', '_for', '_while'}:
                shape = 'diamond'
            elif cnode.ast_node.target.id in {'enter', 'exit'}:
                shape, peripheries = 'oval', '2'
        else:
            shape = 'rectangle'
        graph.node(cnode.i(), "%d: %s" % (lineno, unhack(cnode.source())),
                   shape=shape, peripheries=peripheries)
        for pn in cnode.parents:
            plineno = pn.lineno()
            if hasattr(pn, 'calllink') and pn.calllink > 0 and not hasattr(
                    cnode, 'calleelink'):
                graph.edge(pn.i(), cnode.i(), style='dotted', weight=100)
                continue

            if arcs:
                if (plineno, lineno) in arcs:
                    graph.edge(pn.i(), cnode.i(), color='green')
                elif plineno == lineno and lineno in cov_lines:
                    graph.edge(pn.i(), cnode.i(), color='green')
                # child is exit and parent is covered
                elif hasattr(cnode, 'fn_exit_node') and plineno in cov_lines:
                    graph.edge(pn.i(), cnode.i(), color='green')
                # parent is exit and one of its parents is covered.
                elif hasattr(pn, 'fn_exit_node') and len(
                        set(n.lineno() for n in pn.parents) | cov_lines) > 0:
                    graph.edge(pn.i(), cnode.i(), color='green')
                # child is a callee (has calleelink) and one of the parents is covered.
                elif plineno in cov_lines and hasattr(cnode, 'calleelink'):
                    graph.edge(pn.i(), cnode.i(), color='green')
                else:
                    graph.edge(pn.i(), cnode.i(), color='red')
            else:
                order = {c.i(): i for i, c in enumerate(pn.children)}
                if len(order) < 2:
                    graph.edge(pn.i(), cnode.i())
                else:
                    o = order[cnode.i()]
                    graph.edge(pn.i(), cnode.i(), color=colors[o], label=kind[o])
    return graph 
```

```py
def unhack(v):
    for i in ['if', 'while', 'for', 'elif']:
        v = re.sub(r'^_%s:' % i, '%s:' % i, v)
    return v 
```

### 示例

#### check_triangle

```py
def check_triangle(a, b, c):
    if a == b:
        if a == c:
            if b == c:
                return "Equilateral"
            else:
                return "Isosceles"
        else:
            return "Isosceles"
    else:
        if b != c:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene"
        else:
            return "Isosceles" 
```

```py
import [inspect](https://docs.python.org/3/library/inspect.html) 
```

```py
to_graph(gen_cfg(inspect.getsource(check_triangle))) 
```

<svg width="676pt" height="487pt" viewBox="0.00 0.00 675.50 486.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 482.5)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="259.75" y="-450.32" font-family="Times,serif" font-size="14.00">1: enter: check_triangle(a, b, c)</text></g> <g id="node9" class="node"><title>3</title> <text text-anchor="middle" x="259.75" y="-373.32" font-family="Times,serif" font-size="14.00">2: if: a == b</text></g> <g id="edge7" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="353.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: check_triangle(a, b, c)</text></g> <g id="node3" class="node"><title>6</title> <text text-anchor="middle" x="144.75" y="-92.83" font-family="Times,serif" font-size="14.00">5: return 'Equilateral'</text></g> <g id="edge1" class="edge"><title>6->2</title></g> <g id="node4" class="node"><title>7</title> <text text-anchor="middle" x="287.75" y="-92.83" font-family="Times,serif" font-size="14.00">7: return 'Isosceles'</text></g> <g id="edge2" class="edge"><title>7->2</title></g> <g id="node5" class="node"><title>8</title> <text text-anchor="middle" x="59.75" y="-146.82" font-family="Times,serif" font-size="14.00">9: return 'Isosceles'</text></g> <g id="edge3" class="edge"><title>8->2</title></g> <g id="node6" class="node"><title>11</title> <text text-anchor="middle" x="466.75" y="-92.83" font-family="Times,serif" font-size="14.00">13: return 'Isosceles'</text></g> <g id="edge4" class="edge"><title>11->2</title></g> <g id="node7" class="node"><title>12</title> <text text-anchor="middle" x="607.75" y="-92.83" font-family="Times,serif" font-size="14.00">15: return 'Scalene'</text></g> <g id="edge5" class="edge"><title>12->2</title></g> <g id="node8" class="node"><title>13</title> <text text-anchor="middle" x="369.75" y="-146.82" font-family="Times,serif" font-size="14.00">17: return 'Isosceles'</text></g> <g id="edge6" class="edge"><title>13->2</title></g> <g id="node10" class="node"><title>4</title> <text text-anchor="middle" x="175.75" y="-287.07" font-family="Times,serif" font-size="14.00">3: if: a == c</text></g> <g id="edge8" class="edge"><title>3->4</title> <text text-anchor="middle" x="226.9" y="-330.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node12" class="node"><title>9</title> <text text-anchor="middle" x="368.75" y="-287.07" font-family="Times,serif" font-size="14.00">11: if: b != c</text></g> <g id="edge13" class="edge"><title>3->9</title> <text text-anchor="middle" x="324.53" y="-330.2" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge12" class="edge"><title>4->8</title> <text text-anchor="middle" x="127.31" y="-243.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node11" class="node"><title>5</title> <text text-anchor="middle" x="175.75" y="-200.82" font-family="Times,serif" font-size="14.00">4: if: b == c</text></g> <g id="edge9" class="edge"><title>4->5</title> <text text-anchor="middle" x="179.88" y="-243.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge10" class="edge"><title>5->6</title> <text text-anchor="middle" x="168.96" y="-146.82" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge11" class="edge"><title>5->7</title> <text text-anchor="middle" x="252.08" y="-146.82" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge17" class="edge"><title>9->13</title> <text text-anchor="middle" x="372.85" y="-243.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node13" class="node"><title>10</title> <text text-anchor="middle" x="476.75" y="-200.82" font-family="Times,serif" font-size="14.00">12: if: a == c</text></g> <g id="edge14" class="edge"><title>9->10</title> <text text-anchor="middle" x="433.34" y="-243.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge15" class="edge"><title>10->11</title> <text text-anchor="middle" x="477.36" y="-146.82" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge16" class="edge"><title>10->12</title> <text text-anchor="middle" x="565.39" y="-146.82" font-family="Times,serif" font-size="14.00">F</text></g></g></svg>

#### cgi_decode

注意，我们目前还不支持*增强赋值*：即类似于`+=`的赋值。

```py
def cgi_decode(s):
    hex_values = {
        '0': 0, '1': 1, '2': 2, '3': 3, '4': 4,
        '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
        'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15,
        'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15,
    }

    t = ""
    i = 0
    while i < len(s):
        c = s[i]
        if c == '+':
            t += ' '
        elif c == '%':
            digit_high, digit_low = s[i + 1], s[i + 2]
            i += 2
            if digit_high in hex_values and digit_low in hex_values:
                v = hex_values[digit_high] * 16 + hex_values[digit_low]
                t += chr(v)
            else:
                raise ValueError("Invalid encoding")
        else:
            t += c
        i += 1
    return t 
```

```py
to_graph(gen_cfg(inspect.getsource(cgi_decode))) 
```

<svg width="1022pt" height="1132pt" viewBox="0.00 0.00 1022.00 1132.00" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 1128)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="550" y="-1095.83" font-family="Times,serif" font-size="14.00">1: enter: cgi_decode(s)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="550" y="-1018.83" font-family="Times,serif" font-size="14.00">2: hex_values = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9, 'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15, 'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15}</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="357" y="-636.58" font-family="Times,serif" font-size="14.00">1: exit: cgi_decode(s)</text></g> <g id="node3" class="node"><title>18</title> <text text-anchor="middle" x="406" y="-713.58" font-family="Times,serif" font-size="14.00">26: return t</text></g> <g id="edge1" class="edge"><title>18->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="550" y="-945.83" font-family="Times,serif" font-size="14.00">9: t = ''</text></g> <g id="edge3" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="550" y="-872.83" font-family="Times,serif" font-size="14.00">10: i = 0</text></g> <g id="edge4" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="550" y="-799.83" font-family="Times,serif" font-size="14.00">11: while: i < len(s)</text></g> <g id="edge5" class="edge"><title>5->6</title></g> <g id="edge21" class="edge"><title>6->18</title> <text text-anchor="middle" x="490.37" y="-756.7" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node9" class="node"><title>7</title> <text text-anchor="middle" x="550" y="-713.58" font-family="Times,serif" font-size="14.00">12: c = s[i]</text></g> <g id="edge7" class="edge"><title>6->7</title> <text text-anchor="middle" x="554.12" y="-756.7" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node8" class="node"><title>17</title> <text text-anchor="middle" x="473" y="-11.82" font-family="Times,serif" font-size="14.00">25: i += 1</text></g> <g id="edge6" class="edge"><title>17->6</title></g> <g id="node10" class="node"><title>8</title> <text text-anchor="middle" x="550" y="-636.58" font-family="Times,serif" font-size="14.00">13: if: c == '+'</text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>9</title> <text text-anchor="middle" x="73" y="-492.32" font-family="Times,serif" font-size="14.00">14: t += ' '</text></g> <g id="edge9" class="edge"><title>8->9</title> <text text-anchor="middle" x="415.06" y="-589.45" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node12" class="node"><title>10</title> <text text-anchor="middle" x="550" y="-546.33" font-family="Times,serif" font-size="14.00">15: if: c == '%'</text></g> <g id="edge10" class="edge"><title>8->10</title> <text text-anchor="middle" x="553.75" y="-589.45" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge17" class="edge"><title>9->17</title></g> <g id="node13" class="node"><title>11</title> <text text-anchor="middle" x="375" y="-438.32" font-family="Times,serif" font-size="14.00">16: digit_high, digit_low = (s[i + 1], s[i + 2])</text></g> <g id="edge11" class="edge"><title>10->11</title> <text text-anchor="middle" x="492.52" y="-492.32" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node18" class="node"><title>16</title> <text text-anchor="middle" x="623" y="-384.32" font-family="Times,serif" font-size="14.00">24: t += c</text></g> <g id="edge16" class="edge"><title>10->16</title> <text text-anchor="middle" x="585.57" y="-492.32" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node14" class="node"><title>12</title> <text text-anchor="middle" x="351" y="-330.32" font-family="Times,serif" font-size="14.00">17: i += 2</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>13</title> <text text-anchor="middle" x="338" y="-257.32" font-family="Times,serif" font-size="14.00">18: if: digit_high in hex_values and digit_low in hex_values</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="edge19" class="edge"><title>13->17</title> <text text-anchor="middle" x="313.38" y="-127.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node16" class="node"><title>14</title> <text text-anchor="middle" x="473" y="-171.07" font-family="Times,serif" font-size="14.00">19: v = hex_values[digit_high] * 16 + hex_values[digit_low]</text></g> <g id="edge14" class="edge"><title>13->14</title> <text text-anchor="middle" x="417.71" y="-214.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node17" class="node"><title>15</title> <text text-anchor="middle" x="473" y="-84.83" font-family="Times,serif" font-size="14.00">20: t += chr(v)</text></g> <g id="edge15" class="edge"><title>14->15</title></g> <g id="edge18" class="edge"><title>15->17</title></g> <g id="edge20" class="edge"><title>16->17</title></g></g></svg>

#### gcd

```py
def gcd(a, b):
    if a<b:
        c: int = a
        a: int = b
        b: int = c

    while b != 0 :
        c: int = a
        a: int = b
        b: int = c % b
    return a 
```

```py
to_graph(gen_cfg(inspect.getsource(gcd))) 
```

<svg width="332pt" height="671pt" viewBox="0.00 0.00 332.08 670.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 666.5)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="213.9" y="-634.33" font-family="Times,serif" font-size="14.00">1: enter: gcd(a, b)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="213.9" y="-557.33" font-family="Times,serif" font-size="14.00">2: if: a < b</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="71.9" y="-88.83" font-family="Times,serif" font-size="14.00">1: exit: gcd(a, b)</text></g> <g id="node3" class="node"><title>11</title> <text text-anchor="middle" x="88.9" y="-165.82" font-family="Times,serif" font-size="14.00">11: return a</text></g> <g id="edge1" class="edge"><title>11->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="172.9" y="-471.07" font-family="Times,serif" font-size="14.00">3: c: int = a</text></g> <g id="edge3" class="edge"><title>3->4</title> <text text-anchor="middle" x="199.98" y="-514.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="212.9" y="-252.07" font-family="Times,serif" font-size="14.00">7: while: b != 0</text></g> <g id="edge7" class="edge"><title>3->7</title> <text text-anchor="middle" x="257.65" y="-398.07" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="166.9" y="-398.07" font-family="Times,serif" font-size="14.00">4: a: int = b</text></g> <g id="edge4" class="edge"><title>4->5</title></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="172.9" y="-325.07" font-family="Times,serif" font-size="14.00">5: b: int = c</text></g> <g id="edge5" class="edge"><title>5->6</title></g> <g id="edge6" class="edge"><title>6->7</title></g> <g id="edge12" class="edge"><title>7->11</title> <text text-anchor="middle" x="162.07" y="-208.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node10" class="node"><title>8</title> <text text-anchor="middle" x="212.9" y="-165.82" font-family="Times,serif" font-size="14.00">8: c: int = a</text></g> <g id="edge9" class="edge"><title>7->8</title> <text text-anchor="middle" x="217.02" y="-208.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node9" class="node"><title>10</title> <text text-anchor="middle" x="251.9" y="-11.82" font-family="Times,serif" font-size="14.00">10: b: int = c % b</text></g> <g id="edge8" class="edge"><title>10->7</title></g> <g id="node11" class="node"><title>9</title> <text text-anchor="middle" x="212.9" y="-88.83" font-family="Times,serif" font-size="14.00">9: a: int = b</text></g> <g id="edge10" class="edge"><title>8->9</title></g> <g id="edge11" class="edge"><title>9->10</title></g></g></svg>

```py
def compute_gcd(x, y):
    if x > y:
        small = y
    else:
        small = x
    for i in range(1, small+1):
        if((x % i == 0) and (y % i == 0)):
            gcd = i

    return gcd 
```

```py
to_graph(gen_cfg(inspect.getsource(compute_gcd))) 
```

<svg width="628pt" height="611pt" viewBox="0.00 0.00 627.67 610.75" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 606.75)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="273.72" y="-574.58" font-family="Times,serif" font-size="14.00">1: enter: compute_gcd(x, y)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="273.72" y="-497.57" font-family="Times,serif" font-size="14.00">2: if: x > y</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="106.72" y="-102.08" font-family="Times,serif" font-size="14.00">1: exit: compute_gcd(x, y)</text></g> <g id="node3" class="node"><title>11</title> <text text-anchor="middle" x="137.72" y="-179.07" font-family="Times,serif" font-size="14.00">10: return gcd</text></g> <g id="edge1" class="edge"><title>11->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="223.72" y="-411.32" font-family="Times,serif" font-size="14.00">3: small = y</text></g> <g id="edge3" class="edge"><title>3->4</title> <text text-anchor="middle" x="255.84" y="-454.45" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="324.72" y="-411.32" font-family="Times,serif" font-size="14.00">5: small = x</text></g> <g id="edge4" class="edge"><title>3->5</title> <text text-anchor="middle" x="306.02" y="-454.45" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node7" class="node"><title>6</title> <text text-anchor="middle" x="273.72" y="-338.32" font-family="Times,serif" font-size="14.00">6: __iv = iter(range(1, small + 1))</text></g> <g id="edge5" class="edge"><title>4->6</title></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node8" class="node"><title>7</title> <text text-anchor="middle" x="273.72" y="-265.32" font-family="Times,serif" font-size="14.00">6: for: __iv.__length__hint__() > 0</text></g> <g id="edge7" class="edge"><title>6->7</title></g> <g id="edge13" class="edge"><title>7->11</title> <text text-anchor="middle" x="217.61" y="-222.2" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node11" class="node"><title>8</title> <text text-anchor="middle" x="338.72" y="-179.07" font-family="Times,serif" font-size="14.00">6: i = next(__iv)</text></g> <g id="edge10" class="edge"><title>7->8</title> <text text-anchor="middle" x="314.23" y="-222.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node9" class="node"><title>10</title> <text text-anchor="middle" x="257.72" y="-11.82" font-family="Times,serif" font-size="14.00">8: gcd = i</text></g> <g id="edge8" class="edge"><title>10->7</title></g> <g id="node10" class="node"><title>9</title> <text text-anchor="middle" x="444.72" y="-102.08" font-family="Times,serif" font-size="14.00">7: if: x % i == 0 and y % i == 0</text></g> <g id="edge9" class="edge"><title>9->7</title> <text text-anchor="middle" x="428" y="-179.07" font-family="Times,serif" font-size="14.00">F</text></g> <g id="edge12" class="edge"><title>9->10</title> <text text-anchor="middle" x="360.95" y="-54.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge11" class="edge"><title>8->9</title></g></g></svg>

#### fib

注意，*for 循环*需要额外的处理。虽然我们正确显示了标签，但*比较节点*需要提取。因此，表示并不准确。

```py
def fib(n,):
    ls = [0, 1]
    for i in range(n-2):
        ls.append(ls[-1] + ls[-2])
    return ls 
```

```py
to_graph(gen_cfg(inspect.getsource(fib))) 
```

<svg width="381pt" height="438pt" viewBox="0.00 0.00 380.83 438.25" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 434.25)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="186.42" y="-402.07" font-family="Times,serif" font-size="14.00">1: enter: fib(n)</text></g> <g id="node4" class="node"><title>3</title> <text text-anchor="middle" x="186.42" y="-325.07" font-family="Times,serif" font-size="14.00">2: ls = [0, 1]</text></g> <g id="edge2" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="77.42" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: fib(n)</text></g> <g id="node3" class="node"><title>8</title> <text text-anchor="middle" x="77.42" y="-92.83" font-family="Times,serif" font-size="14.00">5: return ls</text></g> <g id="edge1" class="edge"><title>8->2</title></g> <g id="node5" class="node"><title>4</title> <text text-anchor="middle" x="186.42" y="-252.07" font-family="Times,serif" font-size="14.00">3: __iv = iter(range(n - 2))</text></g> <g id="edge3" class="edge"><title>3->4</title></g> <g id="node6" class="node"><title>5</title> <text text-anchor="middle" x="186.42" y="-179.07" font-family="Times,serif" font-size="14.00">3: for: __iv.__length__hint__() > 0</text></g> <g id="edge4" class="edge"><title>4->5</title></g> <g id="edge8" class="edge"><title>5->8</title> <text text-anchor="middle" x="142.19" y="-135.95" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node8" class="node"><title>6</title> <text text-anchor="middle" x="186.42" y="-92.83" font-family="Times,serif" font-size="14.00">3: i = next(__iv)</text></g> <g id="edge6" class="edge"><title>5->6</title> <text text-anchor="middle" x="190.54" y="-135.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node7" class="node"><title>7</title> <text text-anchor="middle" x="253.42" y="-15.82" font-family="Times,serif" font-size="14.00">4: ls.append(ls[-1] + ls[-2])</text></g> <g id="edge5" class="edge"><title>7->5</title></g> <g id="edge7" class="edge"><title>6->7</title></g></g></svg>

#### quad_solver

```py
def quad_solver(a, b, c):
    discriminant = b² - 4*a*c
    r1, r2 = 0, 0
    i1, i2 = 0, 0
    if discriminant >= 0:
        droot = math.sqrt(discriminant)
        r1 = (-b + droot) / (2*a)
        r2 = (-b - droot) / (2*a)
    else:
        droot = math.sqrt(-1 * discriminant)
        droot_ = droot/(2*a)
        r1, i1 = -b/(2*a), droot_
        r2, i2 = -b/(2*a), -droot_
    if i1 == 0 and i2 == 0:
        return (r1, r2)
    return ((r1,i1), (r2,i2)) 
```

```py
to_graph(gen_cfg(inspect.getsource(quad_solver))) 
```

<svg width="466pt" height="925pt" viewBox="0.00 0.00 465.62 924.50" xmlns:xlink="http://www.w3.org/1999/xlink"><g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 920.5)"><g id="node1" class="node"><title>1</title> <text text-anchor="middle" x="219.75" y="-888.33" font-family="Times,serif" font-size="14.00">1: enter: quad_solver(a, b, c)</text></g> <g id="node5" class="node"><title>3</title> <text text-anchor="middle" x="219.75" y="-811.33" font-family="Times,serif" font-size="14.00">2: discriminant = b ^ 2 - 4 * a * c</text></g> <g id="edge3" class="edge"><title>1->3</title></g> <g id="node2" class="node"><title>2</title> <text text-anchor="middle" x="203.75" y="-15.82" font-family="Times,serif" font-size="14.00">1: exit: quad_solver(a, b, c)</text></g> <g id="node3" class="node"><title>15</title> <text text-anchor="middle" x="125.75" y="-92.83" font-family="Times,serif" font-size="14.00">15: return (r1, r2)</text></g> <g id="edge1" class="edge"><title>15->2</title></g> <g id="node4" class="node"><title>16</title> <text text-anchor="middle" x="282.75" y="-92.83" font-family="Times,serif" font-size="14.00">16: return ((r1, i1), (r2, i2))</text></g> <g id="edge2" class="edge"><title>16->2</title></g> <g id="node6" class="node"><title>4</title> <text text-anchor="middle" x="219.75" y="-738.33" font-family="Times,serif" font-size="14.00">3: r1, r2 = (0, 0)</text></g> <g id="edge4" class="edge"><title>3->4</title></g> <g id="node7" class="node"><title>5</title> <text text-anchor="middle" x="219.75" y="-665.33" font-family="Times,serif" font-size="14.00">4: i1, i2 = (0, 0)</text></g> <g id="edge5" class="edge"><title>4->5</title></g> <g id="node8" class="node"><title>6</title> <text text-anchor="middle" x="219.75" y="-592.33" font-family="Times,serif" font-size="14.00">5: if: discriminant >= 0</text></g> <g id="edge6" class="edge"><title>5->6</title></g> <g id="node9" class="node"><title>7</title> <text text-anchor="middle" x="101.75" y="-506.07" font-family="Times,serif" font-size="14.00">6: droot = math.sqrt(discriminant)</text></g> <g id="edge7" class="edge"><title>6->7</title> <text text-anchor="middle" x="171.94" y="-549.2" font-family="Times,serif" font-size="14.00">T</text></g> <g id="node12" class="node"><title>10</title> <text text-anchor="middle" x="339.75" y="-506.07" font-family="Times,serif" font-size="14.00">10: droot = math.sqrt(-1 * discriminant)</text></g> <g id="edge10" class="edge"><title>6->10</title> <text text-anchor="middle" x="290.68" y="-549.2" font-family="Times,serif" font-size="14.00">F</text></g> <g id="node10" class="node"><title>8</title> <text text-anchor="middle" x="141.75" y="-433.07" font-family="Times,serif" font-size="14.00">7: r1 = (-b + droot) / (2 * a)</text></g> <g id="edge8" class="edge"><title>7->8</title></g> <g id="node11" class="node"><title>9</title> <text text-anchor="middle" x="162.75" y="-306.07" font-family="Times,serif" font-size="14.00">8: r2 = (-b - droot) / (2 * a)</text></g> <g id="edge9" class="edge"><title>8->9</title></g> <g id="node16" class="node"><title>14</title> <text text-anchor="middle" x="203.75" y="-179.07" font-family="Times,serif" font-size="14.00">14: if: i1 == 0 and i2 == 0</text></g> <g id="edge14" class="edge"><title>9->14</title></g> <g id="node13" class="node"><title>11</title> <text text-anchor="middle" x="332.75" y="-433.07" font-family="Times,serif" font-size="14.00">11: droot_ = droot / (2 * a)</text></g> <g id="edge11" class="edge"><title>10->11</title></g> <g id="node14" class="node"><title>12</title> <text text-anchor="middle" x="330.75" y="-360.07" font-family="Times,serif" font-size="14.00">12: r1, i1 = (-b / (2 * a), droot_)</text></g> <g id="edge12" class="edge"><title>11->12</title></g> <g id="node15" class="node"><title>13</title> <text text-anchor="middle" x="321.75" y="-252.07" font-family="Times,serif" font-size="14.00">13: r2, i2 = (-b / (2 * a), -droot_)</text></g> <g id="edge13" class="edge"><title>12->13</title></g> <g id="edge15" class="edge"><title>13->14</title></g> <g id="edge16" class="edge"><title>14->15</title> <text text-anchor="middle" x="173.55" y="-135.95" font-family="Times,serif" font-size="14.00">T</text></g> <g id="edge17" class="edge"><title>14->16</title> <text text-anchor="middle" x="251.73" y="-135.95" font-family="Times,serif" font-size="14.00">F</text></g></g></svg>

## 调用图

### 安装：Pyan 静态调用图提升器

```py
import [os](https://docs.python.org/3/library/os.html) 
```

```py
import [networkx](https://networkx.org/) as nx 
```

### 调用图辅助工具

```py
import [shutil](https://docs.python.org/3/library/shutil.html) 
```

```py
PYAN = 'pyan3' if shutil.which('pyan3') is not None else 'pyan'

if shutil.which(PYAN) is None:
    # If installed from pypi, pyan may still be missing
    os.system('pip install "git+https://github.com/uds-se/pyan#egg=pyan"')
    PYAN = 'pyan3' if shutil.which('pyan3') is not None else 'pyan'

assert shutil.which(PYAN) is not None 
```

```py
def construct_callgraph(code, name="callgraph"):
    file_name = name + ".py"
    with open(file_name, 'w') as f:
        f.write(code)
    cg_file = name + '.dot'
    os.system(f'{PYAN}  {file_name} --uses --defines --colored --grouped --annotated --dot > {cg_file}') 
```

```py
def callgraph(code, name="callgraph"):
    if not os.path.isfile(name + '.dot'):
        construct_callgraph(code, name)
    return Source.from_file(name + '.dot') 
```

```py
def get_callgraph(code, name="callgraph"):
    if not os.path.isfile(name + '.dot'):
        construct_callgraph(code, name)

    # This is deprecated:
    # return nx.drawing.nx_pydot.read_dot(name + '.dot')

    return nx.nx_agraph.read_dot(name + '.dot') 
```

### 示例：迷宫

为了提供一个有意义的示例，你可以轻松地更改代码复杂度和目标位置，我们从提供的字符串形式的迷宫生成迷宫源代码。这个例子大致基于 Felipe Andres Manzano 关于符号执行的旧[博客文章](https://feliam.wordpress.com/2010/10/07/the-symbolic-maze/)（快速致意！）。

你只需将迷宫指定为一个字符串。就像这样。

```py
maze_string = """
+-+-----+
|X|     |
| | --+ |
| |   | |
| +-- | |
|     |#|
+-----+-+
""" 
```

`maze_string`中的每个字符代表一个方块。对于每个方块，生成一个方块函数。

+   如果当前方块是“良性”（` `），则调用对应下一个输入字符（D，U，L，R）的方块函数。意外的输入字符将被忽略。如果没有更多的输入字符，则返回“VALID”以及当前迷宫状态。`

``*   如果当前方块是“陷阱”（`+`,`|`,`-`），则返回“INVALID”以及当前迷宫状态。*   如果当前方块是“目标”（`#`），则返回“SOLVED”以及当前迷宫状态。``

``代码是通过函数 `generate_maze_code` 生成的。`` ```py` ``` def generate_print_maze(maze_string):     return """ def print_maze(out, row, col):  output  = out +"\\n"  c_row = 0  c_col = 0  for c in list(\"\"\"%s\"\"\"):  if c == '\\n':  c_row += 1  c_col = 0  output += "\\n"  else:  if c_row == row and c_col == col: output += "X"  elif c == "X": output += " "  else: output += c  c_col += 1  return output """ % maze_string  ```py    ``` def generate_trap_tile(row, col):     return """ def tile_%d_%d(input, index):  try: HTMLParser().feed(input)  except: pass  return print_maze("INVALID", %d, %d) """ % (row, col, row, col)  ```py    ``` def generate_good_tile(c, row, col):     code = """ def tile_%d_%d(input, index):  if (index == len(input)): return print_maze("VALID", %d, %d)  elif input[index] == 'L': return tile_%d_%d(input, index + 1)  elif input[index] == 'R': return tile_%d_%d(input, index + 1)  elif input[index] == 'U': return tile_%d_%d(input, index + 1)  elif input[index] == 'D': return tile_%d_%d(input, index + 1)  else : return tile_%d_%d(input, index + 1) """ % (row, col, row, col,        row, col - 1,         row, col + 1,         row - 1, col,         row + 1, col,        row, col)      if c == "X":         code += """ def maze(input):  return tile_%d_%d(list(input), 0) """ % (row, col)      return code  ```py    ``` def generate_target_tile(row, col):     return """ def tile_%d_%d(input, index):  return print_maze("SOLVED", %d, %d)  def target_tile():  return "tile_%d_%d" """ % (row, col, row, col, row, col)  ```py    ``` def generate_maze_code(maze, name="maze"):     row = 0     col = 0     code = generate_print_maze(maze)      for c in list(maze):         if c == '\n':             row += 1             col = 0         else:             if c == "-" or c == "+" or c == "|":                 code += generate_trap_tile(row, col)             elif c == " " or c == "X":                 code += generate_good_tile(c, row, col)             elif c == "#":                 code += generate_target_tile(row, col)             else:                 print("无效的迷宫！请尝试另一个。")             col += 1      return code  ```py    Now you can generate the maze code for an arbitrary maze.    ``` maze_code = generate_maze_code(maze_string)  ```py    ``` print_content(maze_code, filename='.py')  ```py    ``` def print_maze(out, row, col):     output  = out +"\n"     c_row = 0     c_col = 0     for c in list(""" +-+-----+ |X|     | | | --+ | | |   | | | +-- | | |     |#| +-----+-+ """):         if c == '\n':             c_row += 1             c_col = 0             output += "\n"         else:             if c_row == row and c_col == col: output += "X"             elif c == "X": output += "  "             else: output += c             c_col += 1     return output  def tile_1_0(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 0)  def tile_1_1(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 1)  def tile_1_2(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 2)  def tile_1_3(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 3)  def tile_1_4(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 4)  def tile_1_5(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 5)  def tile_1_6(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 6)  def tile_1_7(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 7)  def tile_1_8(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 1, 8)  def tile_2_0(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 2, 0)  def tile_2_1(input, index):     if (index == len(input)): return print_maze("VALID", 2, 1)     elif input[index] == 'L': return tile_2_0(input, index + 1)     elif input[index] == 'R': return tile_2_2(input, index + 1)     elif input[index] == 'U': return tile_1_1(input, index + 1)     elif input[index] == 'D': return tile_3_1(input, index + 1)     else : return tile_2_1(input, index + 1)  def maze(input):     return tile_2_1(list(input), 0)  def tile_2_2(input, index):     try: HTMLParser().feed(input)     except: pass     return print_maze("INVALID", 2, 2)  def tile_2_3(input, index):     if (index == len(input)): return print_maze("VALID", 2, 3)     elif input[index] == 'L': return tile_2_2(input, index + 1)     elif input[index] == 'R': return tile_2_4(input, index + 1)     elif input[index] == 'U': return tile_1_3(input, index + 1)     elif input[index] == 'D': return tile_3_3(input, index + 1)     else : return tile_2_3(input, index + 1)  def tile_2_4(input, index):     if (index == len(input)): return print_maze("VALID", 2, 4)     elif input[index] == 'L': return tile_2_3(input, index + 1)     elif input[index] == 'R': return tile_2_5(input, index + 1)     elif input[index] == 'U': return tile_1_4(input, index + 1)     elif input[index] == 'D': return tile_3_4(input, index + 1)     else : return tile_2_4(input, index + 1)  def tile_2_5(input, index):     if (index == len(input)): return print_maze("VALID", 2, 5)     elif input[index] == 'L': return tile_2_4(input, index + 1)     elif input[index] == 'R': return tile_2_6(input, index + 1)     elif input[index] == 'U': return tile_1_5(input, index + 1)     elif input[index] == 'D': return tile_3_5(input, index + 1)     else : return tile_2_5(input, index + 1)  def tile_2_6(input, index):     if (index == len(input)): return print_maze("VALID", 2, 6)     elif input[index] == 'L': return tile_2_5(input, index + 1)     elif input[index] == 'R': return tile_2_7(input, index + 1)     elif input[index] == 'U': return tile_1_6(input, index + 1)     elif input[index] == 'D': return tile_3_6(input, index + 1)     else : return tile_2_6(input, index + 1)  def tile_2_7(input, index):     if (index == len(input)): return print_maze("VALID", 2, 7)     elif input[index] == 'L': return tile_2_6(input, index + 1)     elif input[index] == 'R': return tile_2_8(input, index + 1)     elif input[index] == 'U': return tile_1_7(input, index + 1)     elif input[index] == 'D': return tile_3_7(input, index + 1)     else : return tile_2_7(input, index + 1)  def tile_
