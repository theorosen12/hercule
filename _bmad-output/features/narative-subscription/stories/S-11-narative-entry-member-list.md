# S-11 — NarativeEntryScreen — Liste des membres avec bouton d'ajout en amis

**Epic:** narative-subscription
**Linear:** NAR-460
**Status:** ToDo
**Priority:** High
**Layer:** Mobile
**Depends on:** S-09 (NarativeEntryScreen), S-07 (GQL infra)

---

## Goal

Afficher une liste verticale des membres du narative sur la `NarativeEntryScreen`, avec pour chaque membre un bouton d'ajout en amis reflétant l'état de la relation (pas ami / demande en attente / déjà ami).

---

## Architectural Context [EMBED]

### Decision Log

| # | Decision |
|---|----------|
| D1 | **`friendRequest` est résolu via `@ResolveField`** — aucune modification backend. Il suffit d'ajouter le champ dans la sélection GQL de `getNarativeEntryScreen`. |
| D2 | **`friendRequestStatus` est dérivé dans `NarativeEntry.container.tsx`** — le container transforme le `friendRequest` GQL brut en enum `'none' \| 'pending_sent' \| 'friend'` pour simplifier la logique UI. |
| D3 | **L'utilisateur courant est filtré de la liste** — un membre ne doit pas se voir lui-même avec un bouton d'ajout. Filtrage dans le container via `useCurrentUserContext`. |
| D4 | **État optimiste par membre** — `NarativeMemberListContainer` maintient un `Record<memberId, FriendRequestState>` pour les mises à jour immédiates sans refetch. |
| D5 | **`UserLine` comme base display** — `@nara/evanescent/components/Display/UserLine` avec `RightAction` prop pour le bouton conditionnel. |

### Types & Interfaces

```typescript
// Dérivé dans NarativeEntry.container.tsx
type FriendRequestState = 'none' | 'pending_sent' | 'friend';

// Prop étendue de MemberItem dans NarativeEntryScreen
interface MemberItem {
  id: string;
  firstName: string;
  lastName: string;
  profilePicture: { url: string } | null;
  friendRequestState: FriendRequestState;
}

// NarativeMemberRow props
interface NarativeMemberRowProps {
  member: MemberItem;
  onAddFriend: (memberId: string) => void;
  onCancelRequest: (memberId: string) => void;
  isLoading: boolean;
  testID?: string;
}

// NarativeMemberList props
interface NarativeMemberListProps {
  members: MemberItem[];
  onAddFriend: (memberId: string) => void;
  onCancelRequest: (memberId: string) => void;
  loadingMemberIds: Set<string>;
  testID?: string;
}

// NarativeMemberListContainer props
interface NarativeMemberListContainerProps {
  members: MemberItem[];
  testID?: string;
}
```

### Button Rendering Logic (NarativeMemberRow)

| `friendRequestState` | Bouton |
|----------------------|--------|
| `'none'` | `IconButton` PlusCircle, variant `ghost`, `onPress={onAddFriend}` |
| `'pending_sent'` | `Button` label `"Demande envoyée"`, variant `secondary`, size `small`, `onPress={onCancelRequest}` |
| `'friend'` | `null` |

### FriendRequestState Derivation (NarativeEntry.container.tsx)

```typescript
const deriveFriendRequestState = (
  friendRequest: GqlFriendRequest | null | undefined,
  currentUserId: string,
): FriendRequestState => {
  if (!friendRequest) return 'none';
  if (friendRequest.status === 'accepted') return 'friend';
  if (
    friendRequest.status === 'pending' &&
    friendRequest.requester?.id === currentUserId
  ) return 'pending_sent';
  return 'none';
};
```

---

## Files to Create

- `mobile/src/domains/narative/subscription/view/components/NarativeMemberRow/NarativeMemberRow.tsx`
- `mobile/src/domains/narative/subscription/view/components/NarativeMemberRow/index.ts`
- `mobile/src/domains/narative/subscription/view/components/NarativeMemberList/NarativeMemberList.tsx`
- `mobile/src/domains/narative/subscription/view/components/NarativeMemberList/index.ts`
- `mobile/src/domains/narative/subscription/view/containers/NarativeMemberList/NarativeMemberList.container.tsx`
- `mobile/src/domains/narative/subscription/view/containers/NarativeMemberList/index.ts`

## Files to Modify

- `mobile/src/domains/narative/subscription/infra/queries/getNarativeEntryScreen/getNarativeEntryScreen.query.gql` — ajouter `friendRequest { id status requester { id } }` dans `memberList`
- `mobile/src/domains/narative/subscription/view/screens/NarativeEntryScreen/NarativeEntryScreen.tsx` — étendre `MemberItem`, intégrer `NarativeMemberListContainer`
- `mobile/src/domains/narative/subscription/view/containers/NarativeEntry/NarativeEntry.container.tsx` — ajouter `useCurrentUserContext`, dériver `friendRequestState`, filtrer l'utilisateur courant

> ⚠️ Après modification du `.gql`, lancer `npm run codegen` depuis `mobile/`.

---

## Tasks

- [ ] Modifier `getNarativeEntryScreen.query.gql` — ajouter `friendRequest`
- [ ] Lancer `npm run codegen`
- [ ] Créer `NarativeMemberRow` (pure component, `UserLine` + bouton conditionnel)
- [ ] Créer `NarativeMemberList` (liste verticale de `NarativeMemberRow`)
- [ ] Créer `NarativeMemberListContainer` (mutations + état optimiste)
- [ ] Mettre à jour `MemberItem` dans `NarativeEntryScreen.tsx` + intégrer `NarativeMemberListContainer`
- [ ] Mettre à jour `NarativeEntry.container.tsx` — derive `friendRequestState`, filtre self

---

## Dev Notes

### NarativeMemberRow

```typescript
import { UserLine } from '@nara/evanescent/components/Display/UserLine/UserLine.component';
import { IconButton, Button } from '@nara/evanescent/components/Input/Button/Button.component';
import { PlusCircle } from '@nara/evanescent/components/Icons/PlusCircle.icon';

// RightAction logic:
// state === 'none'         → <IconButton Icon={PlusCircle} variant="ghost" isRounded onPress={...} isDisabled={isLoading} />
// state === 'pending_sent' → <Button label="Demande envoyée" variant="secondary" size="small" onPress={...} isDisabled={isLoading} />
// state === 'friend'       → null
```

### NarativeMemberListContainer

```typescript
// State: Record<string, FriendRequestState> pour overrides optimistes
// Mutations: useSendFriendRequestMutation, useDeletePendingFriendRequestMutation
// Paths:
//   send:   mobile/src/domains/user/profile/infra/mutations/sendFriendRequest/
//   cancel: mobile/src/domains/user/profile/infra/mutations/deletePendingFriendRequest/

const handleAddFriend = async (memberId: string) => {
  setOverrides(prev => ({ ...prev, [memberId]: 'pending_sent' }));
  try {
    await sendFriendRequest({ variables: { requestedUserId: memberId } });
  } catch {
    setOverrides(prev => ({ ...prev, [memberId]: 'none' }));
  }
};

const handleCancelRequest = async (memberId: string) => {
  setOverrides(prev => ({ ...prev, [memberId]: 'none' }));
  try {
    await deletePendingFriendRequest({ variables: { requestedUserId: memberId } });
  } catch {
    setOverrides(prev => ({ ...prev, [memberId]: 'pending_sent' }));
  }
};
```

### NarativeEntry.container.tsx — mapping

```typescript
import { useCurrentUserContext } from '@shared/providers/CurrentUserProvider/CurrentUser.context';

const { user: currentUser } = useCurrentUserContext();

// Dans le mapping memberList:
memberList: screen.memberList
  .filter((m) => m.id !== currentUser.id)
  .map((m) => ({
    id: m.id,
    firstName: m.firstName,
    lastName: m.lastName,
    profilePicture: m.profilePicture ? { url: m.profilePicture.url } : null,
    friendRequestState: deriveFriendRequestState(m.friendRequest, currentUser.id),
  })),
```

### NarativeEntryScreen — intégration

Placer `<NarativeMemberListContainer members={memberList} />` entre `<Heading>` et `<S.NarativeCardWrapper>`.

---

## Edge Cases

- **Liste vide** : ne pas afficher le composant si `memberList.length === 0`
- **Erreur mutation** : rollback optimiste → bouton revient à l'état précédent
- **Loading** : `isLoading` par membre via un `Set<string>` de `loadingMemberIds`
- **Utilisateur courant filtré** : filtrage dans le container, pas dans le composant pure

---

## Acceptance Criteria

- [ ] La liste des membres s'affiche verticalement sur la `NarativeEntryScreen`
- [ ] Chaque membre affiche avatar + prénom + nom via `UserLine`
- [ ] Bouton PlusCircle visible si `friendRequestState === 'none'`
- [ ] Clic PlusCircle → état optimiste `'pending_sent'` → bouton "Demande envoyée"
- [ ] Clic "Demande envoyée" → état optimiste `'none'` → retour à PlusCircle
- [ ] Aucun bouton si `friendRequestState === 'friend'`
- [ ] L'utilisateur courant n'apparaît pas dans la liste
- [ ] Boutons disabled pendant leur mutation respective
- [ ] Rollback si mutation échoue
- [ ] Pas de régression sur le reste de la `NarativeEntryScreen`
