# ActionGUI-secprover

SecProver

SecProver is a tool which implements a methodology to reasoning about fine-grained access control policies (FGAC). This methodology uses SecureUML to specify FGAC policies, a modeling language that extends role-based access control (RBAC) with authorization constraints. The constraints are formalized using the Object Constraint Language (OCL).

The tool secProver is available online. See here.


Current limitations

Data model:

Generalizations between classes are not allowed
Attributes can only have one of the following types: String and Integer and Role.
Association-ends can have multiplicity 0..1 or 0..*
There must be a enumerate called Role, with the roles.
There must be one and only one entity with an attribute called role with type Role.
Security model:

Roles should appears as the same name than in the data model.
OCL expressions:

With respect to types:

Only expressions of class types, Set- and Bag-types over class types, and Set- and Bag-types over String and Integer are allowed.
With respect to operations and iterators:
Operations on String are not allowed, except = and <>.
Operations on Bag-types are not allowed, except ->asSet().
ocIsTypeOf() and oclIsKindOf() are not allowed.
oclAsType() is not allowed.
asSet() is not allowed.
->size() is not allowed.
min() and max() are not allowed.
->sum() is not allowed.
->one() is not allowed.
->isUnique() is not allowed.
->iterate() is not allowed.
->closure() is not allowed.
let is not allowed.
if_then_else is not allowed.

Web application

The tool SecProver is available online here.

User manual
From scratch:
1. Select the DTM-tab, and type-in an ActionGUI data model M in Δ.

2. Select the STM-tab, and type-in an ActionGUI security model M in Δ.

3. Select the SecProver-tab, and add as many OCL boolean expressions (invariants) exp1,..., expn as you want by (repeatedly) clicking on the Add-button and typing-in the desired expression expi.

Notice that you can always modify or remove OCL boolean expressions already typed-in by clicking on the buttons Remove and Modify.

4. Select the OCL to FOL-tab, and add an OCL expression expn+1 (additional constraint) and select the type, by clicking the radio-bottom. Note that the OCL expression should be boolean.

5. Select the information related to the analysis to check: i.e: action, resource, role and type.

6. Select an SMT solver (currently, either Z3 or Yices; or directly SMT2).

7. Generate the environment....  with the syntax of the selected SMT solver.

From a stored project
To store the SecProver project that you have typed in the SecProver web application as an SecProver project, simply click on the button Save and download the generated XML file (by default, memento.xml) to your local disk.

To restore an SecProver project, simply click on the button Load and upload the project from your local disk.


Example

Data model

EmplBasic specifies basic employees’ information. In this example, an employee has a name, a surname, and a salary. An employee may have a supervisor and may in turn supervise other employees. Also, an employee may take one of two roles: “Worker” or “Supervisor”. In the terminology of ComponentUML, Employee is an entity; name, surname, salary, and role are attributes; supervisedBy and supervises are association-ends; and Role is an enumerated class. Notice that the association-end supervises has multiplicity 0..*, meaning that an employee may supervise zero or more employees, while the association-end supervisedBy has multiplicity 0..1 meaning that an employee may have at most one supervisor.
 
enum Role {
    WORKER
    SUPERVISOR
}

entity Employee {
    String name
    String surname
    Integer salary
    Role role

    Set(Employee) supervises oppositeTo supervisedBy
    Employee supervisedBy oppositeTo supervises
}
Security model

The security model specifies a basic FGAC policy for accessing the employees’ information considered in EmplBasic.dtm. In this example, permissions are assigned to users with the role Worker and users with role Supervisor. In addition, users with role Supervisor inherit all the permissions granted to users with role Worker.
In the terminology of SecureUML, Worker and Supervisor are roles, and the former is a superrole of the latter.

role WORKER {
    Employee {
        read(salary) constrainedBy [caller = self]
    }
}

role SUPERVISOR extends WORKER {
    Employee {
        read(salary)
        update(salary) constrainedBy [self.supervisedBy = caller]
    }
}

Invariants

We can refine the model EmplBasic by adding invariants. In particular, we can specify that:
There is exactly one employee who has no supervisor.
Nobody is its own supervisor.
An employee has role Supervisor if and only if it has at least one supervisee.
Every employee has one role.
These invariants can be formalized in OCL as follows:
Employee.allInstances()->one(e| e.supervisedBy.oclIsUndefined())
Employee.allInstances()->forAll(e|not(e.supervisedBy = e))
Employee.allInstances()->forAll(e|(e.role = Supervisor implies e.supervises->notEmpty())
and (e.supervises->notEmpty() implies e.role = Supervisor))
Employee.allInstances()->forAll(e|not(e.role.oclIsUndefined())
Additional Constraints

You can add an additional contraint to the example. In this case we don't add anyone.

Analisys information

The extra information required is an action (i.e. create, read, update, delete),  a resource (i.e. Employee, Employee:name, Employee:surname, etc), select a role (i.e. Supervisor or Worker) and finally a type.  Types are always four: type I, type II, type III, and type IV. Those refer to, consider a role r and an action act, then:

Type I: If there is any scenario in which someone with role r will be allow to execute the action act.
Type II: If there is any scenario in which someone with role r will not be allow to execute the action act.
Type III: If there is any scenario in which nobody with role r will be allow to execute the action act.
Type IV: If, in any scenario, there is at least one object upon which nobody with role r will be allow to execute the action act.
