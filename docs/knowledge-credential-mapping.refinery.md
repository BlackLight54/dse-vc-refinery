```refinery
class Entity {
    contains CredentialEntity[0..10] trace opposite from
}

class Relationship {
	Entity[1] source
    Entity[1] target
    contains CredentialRelationship[1] trace opposite from
}

pred neighbours(Entity s, Entity t) <-> Relationship::source(e, s), Relationship::target(e, t); Relationship::source(e, t), Relationship::target(e, s).

pred transitiveNeigbour(Entity s, Entity t) <-> neighbours+(s, t).

pred loopRelationship(Relationship e) <-> Relationship::source(e, s), Relationship::target(e, s).

error contains_loop_relationship(Relationship e) <-> loopRelationship(e).

error non_connected(Entity n, Entity m) <-> n != m, !transitiveNeigbour(n, m).

%% ------------------------------
%% Verifiable credential
class Credential {
	% contains CredentialType[1] type 
	%    contains Uuid[1] id
	%    contains DateTime[1] validFrom
	%    contains DateTime[1] issueDate
	%    contains Proof[1] proof
	%    contains Did[1] issuer
	contains Data[1] credentialSubject
}

abstract class CredentialField {

}

% class CredentialType extends CredentialField {
% }
% class DateTime extends CredentialField {
% }
% class Uuid extends CredentialField {
% }
% class Proof extends CredentialField {
% }
% class Did extends CredentialField {
% }
class Data extends CredentialField {
	CredentialEntity[1] data
}

% Constraints
error noSubject(Credential c) <-> 
    !credentialSubject(c,_).
error multiple_creds_point_to_same_fact_set(Credential cred1, Credential cred2) <->
    credentialSubject(cred1,subject1),
    credentialSubject(cred2,subject2),
    data(subject1,entity1),
    data(subject2,entity2),
    entity1 != entity2,
    connected_creds(entity1,entity2).
% error multiple_creds_point_to_same_entity(Credential cred1, Credential cred2) <->
%     credentialSubject(cred1,subject1),
%     credentialSubject(cred2,subject2),
%     data(subject1,entity1),
%     data(subject2,entity2),
%     entity1 == entity2.
    
% Instance model
% ----
% trace 
class CredentialEntity {
    container Entity from opposite trace 
}

class CredentialRelationship {
	CredentialEntity[1] source
    CredentialEntity[1] target
    container Relationship from opposite trace
}
pred good_trace(CredentialRelationship cred_rel) <->
    CredentialRelationship::source(cred_rel,cred_source),
    CredentialEntity::from(cred_source, source_traced_from),
    
    CredentialRelationship::target(cred_rel,cred_target),
    CredentialEntity::from(cred_target, target_traced_from),

    CredentialRelationship::from(cred_rel,rel_traced_from),
    Relationship::source(rel_traced_from,source_traced_from ),
    Relationship::target(rel_traced_from,target_traced_from).
error bad_trace(CredentialRelationship cred_rel) <-> !good_trace(cred_rel).
pred cred_entity_has_relationship(CredentialEntity entity, CredentialRelationship rel) <->
    CredentialRelationship::source(rel,entity);
    CredentialRelationship::target(rel,entity).
pred cred_entity_connected(CredentialEntity entity1, CredentialEntity entity2) <->
    entity1 != entity2,
    exists(entity1),
    exists(entity2),
    cred_entity_has_relationship(entity1, rel),
    cred_entity_has_relationship(entity2, rel),
    exists(rel).
error no_source_or_target(CredentialEntity entity) <->
    !cred_entity_has_relationship(entity,_).
pred connected_creds(CredentialEntity entity1, CredentialEntity entity2) <->
    % entity1 != entity2,
    exists(entity1),
    exists(entity2),
    cred_entity_connected+(entity1,entity2).
pred non_connected_creds(CredentialEntity entity1, CredentialEntity entity2) <->
    entity1 != entity2,
    exists(entity1),
    exists(entity2),
    !cred_entity_connected+(entity1,entity2).
% error credential_has_no_data(Credential c) <-> credentialSubject(c, d), data(d, e), !exists(e).

% error relationship_points_outside_credential(Credential c) <-> credentialSubject(c, s), CredentialRelationship::source(r, s), CredentialRelationship::target(r, t), credentialSubject(o, t), o!= c.

% error credential_contains_no_relationship(Credential c) <-> 
%     credentialSubject(c,s),
%     CredentialRelationship::source(r,s),
%     CredentialRelationship::target(r,t),
%     exists(t).
scope node = 5..100, Relationship = 2..4, Entity = 3..5,Credential = 1..3.

```
![[Pasted image 20240322225609.png]]
```refinery
class Entity {
	contains CredentialEntity[0..10] trace opposite from
}

class Relationship {
	Entity[1] source
    Entity[1] target
    contains CredentialRelationship[1] trace opposite from
}

pred neighbours(Entity s, Entity t) <-> Relationship::source(e, s), Relationship::target(e, t); Relationship::source(e, t), Relationship::target(e, s).

pred transitiveNeigbour(Entity s, Entity t) <-> neighbours+(s, t).

pred loopRelationship(Relationship e) <-> Relationship::source(e, s), Relationship::target(e, s).

error contains_loop_relationship(Relationship e) <-> loopRelationship(e).

error non_connected(Entity n, Entity m) <-> n != m, !transitiveNeigbour(n, m).

%% ------------------------------
%% Verifiable credential
class Credential {
	% contains CredentialType[1] type 
	%    contains Uuid[1] id
	%    contains DateTime[1] validFrom
	%    contains DateTime[1] issueDate
	%    contains Proof[1] proof
	%    contains Did[1] issuer
	contains Data[1] credentialSubject
}

abstract class CredentialField {

}

% class CredentialType extends CredentialField {
% }
% class DateTime extends CredentialField {
% }
% class Uuid extends CredentialField {
% }
% class Proof extends CredentialField {
% }
% class Did extends CredentialField {
% }
class Data {
	CredentialEntity[1] data
}

% Constraints
error noSubject(Credential c) <-> !credentialSubject(c, _).

error multiple_creds_point_to_same_fact_set(Credential cred1, Credential cred2) <-> credentialSubject(cred1, subject1), credentialSubject(cred2, subject2), data(subject1, entity1), data(subject2, entity2), entity1 != entity2, connected_creds(entity1, entity2).

% error multiple_creds_point_to_same_entity(Credential cred1, Credential cred2) <->
%     credentialSubject(cred1,subject1),
%     credentialSubject(cred2,subject2),
%     data(subject1,entity1),
%     data(subject2,entity2),
%     entity1 == entity2.
% Instance model
% ----
% trace 
class CredentialEntity {
	container Entity from opposite trace
}

class CredentialRelationship {
	CredentialEntity[1] source
    CredentialEntity[1] target
    container Relationship from opposite trace
}

pred good_trace(CredentialRelationship cred_rel) <-> CredentialRelationship::source(cred_rel, cred_source), CredentialEntity::from(cred_source, source_traced_from), CredentialRelationship::target(cred_rel, cred_target), CredentialEntity::from(cred_target, target_traced_from), CredentialRelationship::from(cred_rel, rel_traced_from), Relationship::source(rel_traced_from, source_traced_from), Relationship::target(rel_traced_from, target_traced_from).

error bad_trace(CredentialRelationship cred_rel) <-> !good_trace(cred_rel).

pred cred_entity_has_relationship(CredentialEntity entity, CredentialRelationship rel) <-> CredentialRelationship::source(rel, entity); CredentialRelationship::target(rel, entity).

pred cred_entity_connected(CredentialEntity entity1, CredentialEntity entity2) <-> entity1 != entity2, exists(entity1), exists(entity2), cred_entity_has_relationship(entity1, rel), cred_entity_has_relationship(entity2, rel), exists(rel).

error no_source_or_target(CredentialEntity entity) <-> !cred_entity_has_relationship(entity, _).

pred connected_creds(CredentialEntity entity1, CredentialEntity entity2) <-> % entity1 != entity2,
exists(entity1), exists(entity2), cred_entity_connected+(entity1, entity2).

pred non_connected_creds(CredentialEntity entity1, CredentialEntity entity2) <-> entity1 != entity2, exists(entity1), exists(entity2), !cred_entity_connected+(entity1, entity2).

% error credential_has_no_data(Credential c) <-> credentialSubject(c, d), data(d, e), !exists(e).
% error relationship_points_outside_credential(Credential c) <-> credentialSubject(c, s), CredentialRelationship::source(r, s), CredentialRelationship::target(r, t), credentialSubject(o, t), o!= c.
% error credential_contains_no_relationship(Credential c) <-> 
%     credentialSubject(c,s),
%     CredentialRelationship::source(r,s),
%     CredentialRelationship::target(r,t),
%     exists(t).
scope node = 5..100, Relationship = 2, Entity = 3..5,Credential = 2, CredentialEntity = 4.

```
![[Pasted image 20240323091333.png]]

```refinery	
class Entity {
	contains CredentialEntity[0..10] trace opposite from
}

class Relationship {
	Entity[1] IGsource
    Entity[1] IGtarget
    contains CredentialRelationship[1] trace opposite from
}

pred neighbours(Entity s, Entity t) <-> IGsource(e, s), IGtarget(e, t); IGsource(e, t), IGtarget(e, s).

pred transitiveNeigbour(Entity s, Entity t) <-> neighbours+(s, t).

pred loopRelationship(Relationship e) <-> IGsource(e, s), IGtarget(e, s).

error contains_loop_relationship(Relationship e) <-> loopRelationship(e).

error non_connected(Entity n, Entity m) <-> n != m, !transitiveNeigbour(n, m).

%% ------------------------------
%% Verifiable credential
class Credential {
	% contains CredentialType[1] type 
	%    contains Uuid[1] id
	%    contains DateTime[1] validFrom
	%    contains DateTime[1] issueDate
	%    contains Proof[1] proof
	%    contains Did[1] issuer
	contains Data[1] data
}

abstract class CredentialField {

}

% class CredentialType extends CredentialField {
% }
% class DateTime extends CredentialField {
% }
% class Uuid extends CredentialField {
% }
% class Proof extends CredentialField {
% }
% class Did extends CredentialField {
% }
class Data {
	CredentialEntity[1] credentialSubject
}

% Constraints
error noSubject(Credential c) <-> !data(c, _).

error multiple_creds_point_to_same_fact_set(Credential cred1, Credential cred2) <-> data(cred1, subject1), data(cred2, subject2), credentialSubject(subject1, entity1), credentialSubject(subject2, entity2), entity1 != entity2, connected_creds(entity1, entity2).

% error multiple_creds_point_to_same_entity(Credential cred1, Credential cred2) <->
%     data(cred1,subject1),
%     data(cred2,subject2),
%     credentialSubject(subject1,entity1),
%     credentialSubject(subject2,entity2),
%     entity1 == entity2.
% Instance model
% ----
% trace 
class CredentialEntity {
	container Entity from opposite trace
}

class CredentialRelationship {
	CredentialEntity[1] credSource
    CredentialEntity[1] credTarget
    container Relationship from opposite trace
}

pred good_trace(CredentialRelationship cred_rel) <-> credSource(cred_rel, cred_source), CredentialEntity::from(cred_source, source_traced_from), credTarget(cred_rel, cred_target), CredentialEntity::from(cred_target, target_traced_from), CredentialRelationship::from(cred_rel, rel_traced_from), IGsource(rel_traced_from, source_traced_from), IGtarget(rel_traced_from, target_traced_from).

error bad_trace(CredentialRelationship cred_rel) <-> !good_trace(cred_rel).

pred cred_entity_has_relationship(CredentialEntity entity, CredentialRelationship rel) <-> credSource(rel, entity); credTarget(rel, entity).

pred cred_entity_connected(CredentialEntity entity1, CredentialEntity entity2) <-> entity1 != entity2, exists(entity1), exists(entity2), cred_entity_has_relationship(entity1, rel), cred_entity_has_relationship(entity2, rel), exists(rel).

error no_source_or_target(CredentialEntity entity) <-> !cred_entity_has_relationship(entity, _).

pred connected_creds(CredentialEntity entity1, CredentialEntity entity2) <-> % entity1 != entity2,
exists(entity1), exists(entity2), cred_entity_connected+(entity1, entity2).

pred non_connected_creds(CredentialEntity entity1, CredentialEntity entity2) <-> entity1 != entity2, exists(entity1), exists(entity2), !cred_entity_connected+(entity1, entity2).

% error credential_has_no_credentialSubject(Credential c) <-> data(c, d), credentialSubject(d, e), !exists(e).
% error relationship_points_outside_credential(Credential c) <-> data(c, s), CredentialRelationship::source(r, s), CredentialRelationship::target(r, t), data(o, t), o!= c.
% error credential_contains_no_relationship(Credential c) <-> 
%     data(c,s),
%     CredentialRelationship::source(r,s),
%     CredentialRelationship::target(r,t),
%     exists(t).
scope node = 5..100, Relationship = 2, Entity = 3..5,Credential = 2, CredentialEntity = 4.
```