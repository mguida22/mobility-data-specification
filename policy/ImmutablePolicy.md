# Mobility Data Specification: Policy and Immutable Data

The MDS Policy specification contains a note as to how a given Policy is updated:

> Published policies, like geographies, should be treated as immutable data. Obsoleting or otherwise changing a policy is 
> accomplished by publishing a new policy with a field named `prev_policies`, a list of UUID references to the policy or 
> policies superseded by the new policy.

This raises a number of questions about the practical implications of using this immutable data structure.  

**Q:** Why are Policy objects immutable? <br/>
**A:** Policies may be used by cities for enforcement actions such as fees, fines, and permit suspensions.  An unchangable history of all published policies makes this possible.  Updating a Policy on the fly would potentially be re-writing history.

**Q:** But `end_date` is nullable.  Wouldn't a null `end_date` make a Policy object in effect forever? <br/>
**A:** No, a Policy object can be updated or terminated by a publishing a subsequent Policy object.

**Q:** How do I end a Policy which has a null `end_date`, or end a Policy before its `end_date` if needed? <br/>
**A:** Publish a new Policy with a `prev_policies` list that includes the terminated Policy's `policy_id`.  A Policy with an empty list of Rules will terminate the prior policy.

**Q:** Does publishing a supercding policy immediately update or terminate the old policy? <br/>
**A:** Not necessarily; the new policy becomes effective as of its `start_date`, effectively overwriting the `end_date` of the old policy.  That start date can be in the future, and in practice, *should* be in the future to give the consumers a chance to plan.

**Q:** If I publish a superceding empty policy with a non-null `end_date`, would that simply suppress the superceded policy until the new policy's `end_date`, and then the original policy would become effective again? <br/>
**A:** In the interest of simplicity, the official stance of the OMF at present is 'no.'  A superceded Policy cannot be brought back to life.

**Q:** How much lead-time is required for obsoleting or updating a Policy? <br/>
**A:** This is outside of the spec and should be addressed in an SLA.  Generally cities and other regulatory agencies should allow ample lead-time whenever possible, as fully-automated consumption of MDS Policy has yet to be implemented by any Provider to the best of our knowledge.  OTOH one could imagine emergency situations such as a natural disaster where a Policy might need to be updated as soon as humanly possible.

**Q:** Why not make just `end_date` immutable? <br/>
**A**: Picking-and-choosing which fields are immutable seems perilous, and this document should make it easier to understand the way to reason about immutable policy data without resorting to short-cuts.

**Q:** Have you thought about digitially signing Policy objects to make sure that they haven't changed, and that the publisher is verifyable? <br/>
**A:** It has come up in discussions, but thus far no action has been taken.  No standard for digitally signing JSON has been established, as JSON does not specify a field-ordering within objects.

**Q**: What if I accidentally terminate a Policy?  How do I un-terminate it? <br/>
**A**: Re-publish the original Policy with a different `policy_id` and a `prev_policy` list containing the terminating (empty) Policy if you want to make sure the connection to the original is explicit, otherwise just publish a new policy and don't sweat the `prev_policy` field.

**Q**: What if all Policy objects were on a blockchain? <br/>
**A**: No.